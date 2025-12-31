---
layout: "post"
title: "Додавання статичної IP адреси віртуальним машинам Multipass на macOS"
date: "2025-12-26 08:30:00 +0200"
categories: uk
tags:
  - DevOps
  - Infrastructure
  - Multipass
  - cloud-init
  - Netplan
comments: true
lang: uk
ref: 2025-12-26-multipass-vm-static-ip
---

Цей посібник містить докладний опис того як додати статичну IP адресу до віртуальної машини Multipass на macOS. Загальний опис того як це зробити ви можете знайти в офіційній документації в розділі [Configure static IPs](https://documentation.ubuntu.com/multipass/latest/how-to-guides/manage-instances/configure-static-ips/), однак на macOS без певних модифікацій повторити ці рекомендації не виходить.

<iframe width="560" height="315" src="https://www.youtube.com/embed/X_PqiMvaE08?si=1JVrvxjHMduGariC" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<a id="default-eth"></a>

## Додавання IP адреси до першого мережевого інтерфейсу віртуальної машини

### Пошук Multipass bridge

Для доступу до віртуальних машин Multipass (на macOS) створює міст з назвою `bridge100`. Цей міст використовується для доступу віртуальних машин до мережі хоста. IP адреси призначаються через DHCP і при (пере)створенні віртуальної машини їй надається наступна IP адреса з мережі моста.

Виконання команди `multipass networks` має показати вам перелік доступних мережевих інтерфейсів.

![multipass networks](/images/2025/12/2025-12-26-multipass-networks.png)

Як ви можете бачити в цьому переліку нашого моста немає. Результат `multipass networks` підтверджує головне обмеження Multipass на macOS: він "бачить" лише фізичні мережеві адаптери (Wi-Fi, Ethernet, USB). Створені віртуальні мости (bridge100) він ігнорує, оскільки вони не мають апаратного профілю, який очікує драйвер віртуалізації Apple.

Спробуємо знайти міст іншим шляхом.

```bash
# Знайдіть bridge (зазвичай bridge100)
ifconfig | grep -A 2 "^bridge"
```

Відподіь має бути схожа на наступне:

![](/images/2025/12/2025-12-26-grep-bridge.png)

**Запамʼятайте назву bridge** (наприклад, `bridge100`). Ми будема далі її використовувати.

В моєму випадку міст доступний за адресою `192.168.2.1/24` і DHCP сервер виділяє адреси віртуальним машинам з діапазону `192.168.2.X`.

`ifconfig -v bridge100`

![](/images/2025/12/2025-12-26-ifconfig-bridge100.png)

### Налаштування bridge на хості

Припустимо, що нам потрібно надавати віртуальним машинам статичні IP адреси для мережі `10.10.0.0/24`. Для цього ми будемо створювати аліас до наявного у нас мосту.

```bash
# Додайте IP адресу до bridge
sudo ifconfig bridge100 10.10.0.1/24 alias

# Перевірте що додалось
ifconfig bridge100 | grep "inet "
```

**Очікуваний результат:**

```console
inet 192.168.2.1 netmask 0xffffff00 broadcast 192.168.2.255
inet 10.10.0.1 netmask 0xffffff00 broadcast 10.10.0.255
```

![bridge100 alias](/images/2025/12/2025-12-26-bridge100-alias.png)

або за допомогою скрипта

```bash
TARGET_BRIDGE=$(ifconfig -v | grep -B 20 "member: vmenet" | grep "bridge" | awk -F: '{print $1}' | head -n 1)

if [ -z "$TARGET_BRIDGE" ]; then
    echo "Помилка: міст не знайдено. Перевірте, чи запущена ВМ."
else
    echo "ВМ знайдена на $TARGET_BRIDGE. Призначаємо 10.10.0.1..."
    sudo ifconfig $TARGET_BRIDGE 10.10.0.1/24 alias
fi
```

### Створення cloud-init конфігурації для VM

Створимо наступний конфігураційний файл cloud-init, що містить налаштування для мережі `10.10.0.0/24`

```bash
cat > multipass-static-ip.yaml << 'EOF'
#cloud-config

write_files:
  - path: /etc/netplan/60-static-ip.yaml
    permissions: '0600'
    content: |
      network:
        version: 2
        ethernets:
          default:
            dhcp4: true
            addresses:
              - 10.10.0.10/24
            routes:
              - to: default
                via: 10.10.0.1
                metric: 200

runcmd:
  - netplan apply

hostname: test-vm
EOF
```

Для налаштування роботи мережі в Ubuntu використовується [Netplan](https://netplan.io), ми додаємо настройки для нього у файл `/etc/netplan/60-static-ip.yaml`. Вміст файлу знаходиться у полі `content:` Зверніть увагу на рядок `permissions: '0600'`, ним ми встановлюємо права достпу на читання-запис тільки для користувача root. За наявності занадто дозвільних прав Netplan сповістить про це і не застосує налаштування. Команда `netplan apply` застосовує налаштування з теки `/etc/netplan/`. Поле `hostname` містить назву для нашої віртуальної машини, яка буде додано у файл `/etc/hostname`.

### Запуск VM та перевірка роботи

Створимо нашу віртуальну машину

```bash
# Створіть VM з cloud-init конфігурацією
multipass launch --name test-vm --cloud-init multipass-static-ip.yaml
```

Multipass створить віртуальну машину з назвою вказаною в параметрі `--name/-n` та використає налаштування [`cloud-init`](https://cloud-init.io) з файла, вказаного в `--cloud-init`.

Перевіримо мережеві налаштування нашої віртуальної машини

```bash
# Перевірте IP адреси на VM
multipass exec -n test-vm -- ip addr show enp0s1

# або скористайтесь netplan
multipass exec -n test-vm -- netplan status

# Має показати два IP:
# - 192.168.2.x (DHCP)
# - 10.10.0.10 (static)
```

![test-vm netplan status](/images/2025/12/2025-12-26-test-vm-netplan-status.png)

Тепер настав час переврити звʼязок

```bash
# З хоста до VM
ping -c 4 10.10.0.10
```

![ping з хоста до VM](/images/2025/12/2025-12-26-test-vm-ping-from-host.png)

```bash
# З VM до хоста
multipass exec -n test-vm -- ping -c 4 10.10.0.1
```

![ping з VM до хоста](/images/2025/12/2025-12-26-test-vm-ping-from-vm-to-host.png)

```bash
# Перевірте доступ в інтернет з VM
multipass exec -n test-vm -- ping -c 4 8.8.8.8
```

![ping в інтернет з VM](/images/2025/12/2025-12-26-test-vm-ping-from-vm-to-internet.png)

✅ **Статична IP адреса працює!**

### Технічні деталі

```bash
# Вивід переліку файлів конфігурації
multipass exec -n test-vm -- sudo ls -la /etc/netplan

# Отримання обʼєднаної конфігурації мережі
multipass exec -n test-vm -- sudo netplan get
```

Ми бачимо що у теці `/etc/netplan` знаходяться файли `50-cloud-init.yaml`, який створюється cloud-init під час ініціалізації віртуальної машини, та файл `60-static-ip.yaml`, який ми передали в налаштуваннях

Виконання команди `sudo netplan get` надасть нам обʼєднану конфігурацію мережі. Порівняйте її з вмістом файлу `50-cloud-init.yaml`

```bash
# Отримайте вміст 50-cloud-init.yaml
multipass exec -n test-vm -- sudo cat /etc/netplan/50-cloud-init.yaml
```

Зверніть увагу на назву мережевого інтерйфесу що його використовує `cloud-init`. Саме його ми використовували в наших налаштуваннях (не `enp0s1`).

```yaml
network:
  version: 2
  ethernets:
    default: # назва мережевого інтерфейсу
      match:
        macaddress: "52:54:00:ae:24:22"
      dhcp-identifier: "mac"
      dhcp4: true
```

<a id="enp0s2"></a>

## Додавання статичної IP адреси до другого мережевого інтерфейсу віртуальної машини

Окрім додавання статчної IP адреси до першого мережевого інтерфейсу ми можемо робити це й для інших мережевих інтерфейсів вірутальної машини. Для цього на хості потрібно додати/створити відповідний мережевий міст.

### Multipass bridge для другого мережевого інтерфейсу

Запустимо тимчасову ВМ для створення нового моста (bridge101)

```bash
multipass launch --name sandbox-vm --network name=en0,mode=manual
```

Параметр `--network name=en0,mode=manual` створить новий мережевий інтерфейс `enp0s2` у віртуальній машині, який буде привʼзаний до нового bridge. Однак, після створення цей інтерфейс буде неактивним, оскільки йому не було призначено IP-адресу.

Недоліком чи перевагою такого підходу є те, що після видалення всіх віртуальних машин, привʼязаних до цього bridge його буде видалено з системи автоматично. Він існує допоки є віртуальні машини привʼязані до нього.

Ця команда дозволяє дізнатись назву цього bridge:

```bash
ifconfig -v | grep -B 20 "member: vmenet" | grep "bridge" | awk -F: '{print $1}' | tail -n 1
```

Скоріш за все назва буде `bridge101`.

Додамо аліас до мосту

```bash
# Додайте IP адресу до bridge
sudo ifconfig bridge101 10.10.1.1/24 alias

# Перевірте що додалось
ifconfig bridge101 | grep "inet "
```

**Очікуваний результат:**

```console
inet 10.10.1.1 netmask 0xffffff00 broadcast 10.10.1.255
```

![bridge101 alias](/images/2025/12/2025-12-26-bridge101-alias.png)

або

```bash
TARGET_BRIDGE=$(ifconfig -v | grep -B 20 "member: vmenet" | grep "bridge" | awk -F: '{print $1}' | tail -n 1)

if [ -z "$TARGET_BRIDGE" ]; then
    echo "Помилка: міст не знайдено. Перевірте, чи запущена ВМ."
else
    echo "ВМ знайдена на $TARGET_BRIDGE. Призначаємо 10.10.1.1..."
    sudo ifconfig $TARGET_BRIDGE 10.10.1.1/24 alias
fi
```

### Створення cloud-init конфігурації для другого мережевого інтерфейсу VM

Так само, як і в першому випадку створимо конфігурацію cloud-init для налаштування мережевого інтерфейсу віртуальної машини.

```bash
cat > multipass-static-ip1.yaml << 'EOF'
#cloud-config

write_files:
  - path: /etc/netplan/60-custom-network.yaml
    permissions: '0600'
    content: |
      network:
        version: 2
        ethernets:
          enp0s2:
            addresses:
              - 10.10.1.20/24
            # МИ ВИДАЛЯЄМО "via: 10.10.0.1" (default gateway)
            # Замість цього просто дозволяємо прямий доступ до мережі 10.10.1.0/24
            routes:
              - to: 10.10.1.0/24
                scope: link
runcmd:
  - netplan apply
EOF
```

### Запуск та перевірка роботи віртуальної машини

Запустимо віртуальну машину з назвою `test-vm1`.

```bash
# Створіть VM з cloud-init конфігурацією
multipass launch --name test-vm1 --network name=en0,mode=manual --cloud-init multipass-static-ip1.yaml
```

Дочекаємось завершення процесу створення та запуску VM. Після цього ми можемо видалити нашу тимчасову віртуальну машину, яку ми використовували для того, щоб система створила новий мережевий міст.

```bash
multipass delete sandbox-vm --purge
```

Створення віртуальної машини `test-vm1` відбувається так само як і `test-vm` з тією відмінністю, що вона матиме два мережевих інтерфеси.

Тепер перевіримо налаштування мережевих інтерфейсів віртуальної машини

```bash
# Перевірте IP адреси на VM
multipass exec -n test-vm1 -- ip addr show

# або скористайтесь netplan
multipass exec -n test-vm1 -- netplan status
```

Ви маєте побачити два IP:

- 192.168.2.x (DHCP) на enp0s1
- 10.10.1.20 (static) на enp0s2

![test-vm1 netplan status](/images/2025/12/2025-12-26-test-vm1-netplan-status.png)

Перевіримо роботу звʼязку

```bash
# З хоста до VM1
ping -c 4 10.10.1.20
```

![ping з хоста до VM1](/images/2025/12/2025-12-26-test-vm1-ping-from-host.png)

```bash
# З VM1 до хоста
multipass exec -n test-vm1 -- ping -c 4 10.10.1.1
```

![ping з VM1 до хоста](/images/2025/12/2025-12-26-test-vm1-ping-from-vm-to-host.png)

```bash
# Трафік між VM та VM1
multipass exec -n test-vm1 -- ping -c 4 10.10.0.10
multipass exec -n test-vm -- ping -c 4 10.10.1.20
```

![Трафік між VM та VM1](/images/2025/12/2025-12-26-test-vm1-ping-from-vm1-to-vm.png)

```bash
# Перевірте доступ в інтернет з VM1
multipass exec -n test-vm1 -- ping -c 4 8.8.8.8
```

![Перевірте доступ в інтернет з VM1](/images/2025/12/2025-12-26-test-vm-ping-from1-vm-to-internet.png)

✅ **Статична IP адреса на другому мережевому інтерфейсі працює!**

## Очищення

Видаліть віртуальні машини командою `multipass delete <name-vm> --purge` або всі разом — `multipass delete --all --purge`.

Після видалення всіх віртуальних машин, які були привʼязані до мосту `bridge101` його буде вилучено автоматично.

Рузультатом виконання `ifconfig -v bridge101` буде

```console
ifconfig: interface bridge101 does not exist
```

Для видалення аліасу з `bridge100` виконайте `sudo ifconfig bridge100 -alias 10.10.0.1`, також можна очистити кеш `sudo arp -d -a`.

## Підсумки

За допомогою цих настанов ви маєте можливість додавати статичні адреси віртуальним машинам Multipass. Це може бути корисно у випадках коли вам потрібно використовувати пул заздалегідь виділених адрес.

⚠️ Це керівництво було створено та протестовано для роботи на macOS. Робота з іншими операційними системами може мати відмінності, залежно від функцій та підходів, які ви будете використовувати.
