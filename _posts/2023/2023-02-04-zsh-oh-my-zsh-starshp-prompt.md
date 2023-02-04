---
layout: post
title: Zsh + Oh My Zsh + Spaceship prompt
date: '2023-02-04 12:00:00 +0200'
categories:
  - uk
tags:
  - macOS
  - zsh
  - starship
comments: true
lang: uk
ref: zsh-omz-spaceship
---
**Terminal** — це така частина вашої операційної системи, про наявність якої багато хто з пересічних користувачів навіть не здогадується. Однак, якщо ви розробляєте, розгортаєте локально чи віддалено, програмне забезпечення, Terminal стає вашим повсякденним інструментом. Правильне налаштування інструменту допомагає вам виконувати роботу швидше та якісніше.

Тут я хочу поділитись своїми налаштуваннями, щоб в майбутньому самому швидше виконати налаштування на новій системі, а також допомогти вам зробити це самотужки.

- [Zsh](http://zsh.sourceforge.net/) — Z shell, є типовим в багатьох Linux збірках та в macOS
- [Oh-My-Zsh](https://ohmyz.sh/) — інструмент, який дозволяє вам керувати налаштуваннями Zsh в зручний спосіб.
- [Spaceship prompt](https://spaceship-prompt.sh) — запрошення командного рядка для будь-якого шелу, що гнучко налаштовується!

Сподіваюсь ви встановили собі [Homebrew](https://brew.sh) — зручний менеджер керування застосунками для macOS. Користувачі Linux мають такі менеджери «із коробки».

## Встановлення / оновлення Zsh

Спочатку встановимо (оновимо) Zsh.

```sh
brew install zsh
```

Або оновимо, за потреби

```sh
brew upgrade zsh
```

> Див. [Оновлюємо версію zsh у Ventura 13.2 на MacBook pro]({% post_url 2023/2023-02-03-m1-ventura-zsh %})

Якщо ваш стандартний шел був іншим замініть його

```sh
chsh -s $(which zsh)
```

та перезавантажте шел, щоб зміни вступили в дію

```sh
source ~/.zshrc
```

Ви можете вийти з термінала <kbd>⌘</kbd>+<kbd>Q</kbd> та запустити його знову.

Перевірте що ви тепер використовуєте саме Zsh

```sh
echo $0
```

![розташування zsh](/images/2023/02/zsh-check.png)

Бачимо, що **Zsh** є також в заголовку вікна термінала. Команда `which zsh` показує шлях до місця розташування виконавчого файлу; `where zsh` — де знаходяться всі варіанти Zsh.

## Встановлюємо Oh My Zsh

Після оновлення Zsh перейдемо до встановлення [Oh My Zsh](https://ohmyz.sh). Oh My Zsh дозволяє створити шаблон файлу налаштувань `.zshrc` для Zsh.

<https://ohmyz.sh/#install> – містить інструкції щодо встановлення.

```sh
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

Якщо у вас, як у мене досі не має `wget`, встановіть його

```sh
brew install wget
```

Після встановлення Oh My Zsh запрошення командного рядка змініться і стане зеленою тильдою «`~`». Oh My Zsh надасть вам шаблон файлу налаштувань `.zshrc`, де ви можете налаштувати поведінку Zsh, увімкнути додаткові втулки та налаштувати вигляд запрошення командного рядка.

![omz-white](/images/2023/02/omz-white.png)

### Налаштування **`~/.zshrc`**

Налаштуємо шляхи **`$PATH`**

```sh
export PATH=$HOME/bin:/usr/local/bin:/usr/local/sbin:$PATH
```

Додамо автодоповнення для команд. Встановимо `zsh-completion`.

```sh
brew install zsh-completion
```

![install brew zsh-completion](/images/2023/02/brew-zsh-completion.png)

Відповідно до порад додамо наступний код до `~/.zshrc`, щоб увімкнути автодоповнення команд в терміналі.

```sh
  if type brew &>/dev/null; then
    FPATH=$(brew --prefix)/share/zsh-completions:$FPATH

    autoload -Uz compinit
    compinit
  fi
```

Виконаємо примусове перебудування `zcompdump`.

```sh
rm -f ~/.zcompdump; compinit
```

Однак маємо проблему при спробі завантажити код для автодоповнення команд `"zsh compinit: insecure directories"`, виконаємо команду з настанов встановлювача:

```sh
chmod -R go-w '/opt/homebrew/share/zsh'
```

Але все одно при спробі ініціалізації автодоповнювача отримуємо повідомлення про недостатні повноваження

![zsh-completion permissions](/images/2023/02/zsh-completion-permissions.png)

для надання відповідних дозволів виконаємо команду

```sh
compaudit | xargs chmod g-w, o-w
```

та перезавантажимо `~/.zshrc`. Тепер здається все працює так, як очікувалось, принаймні у нас немає повідомлень про помилку чи відсутність дозволів.

Додамо автозавершення команд для Homebrew

```sh
  if type brew &>/dev/null
  then
    FPATH="$(brew --prefix)/share/zsh/site-functions:${FPATH}"

    autoload -Uz compinit
    compinit
  fi
```

Ось так це виглядає у `~/.zshrc`.

![~/.zshrc completion section](/images/2023/02/zshrc-completion.png)

Для налаштування автодоповнення для інших команд знайдіть їх через

```sh
brew search completion
```

та виконайте рекомендації по їх додаванню до `~/.zshrc`.

![brew search completion](/images/2023/02/brew-search-completion.png)

Додайте інші налаштування шляхів для Ruby, Python, Java, nvm та інше до секції ініціалізації Oh My Zsh.

```sh
# Path to your oh-my-zsh installation.
export ZSH="$HOME/.oh-my-zsh"
```

### Підсвічування синтаксису

Встановимо `pygments` для підсвічування синтаксису командою 

```sh
brew install pygments
```

та додамо в `~/.zshrc`

```sh
ZSH_COLORIZE_TOOL=pygmentize
ZSH_COLORIZE_STYLE="default"
```

перед секцією додавання втулків.

### Втулки

Для додавання додаткових можливостей в Oh My Zsh передбачено використання втулків. Мій вибір:

```sh
plugins=(
        git
        nvm
        colored-man-pages
        colorize
        zsh-syntax-highlighting
        zsh-autosuggestions
        zsh-aliases-lsd
)
```

Для `colorize` ми встановили `pygments`, а для `zsh-aliases-lsd` встановимо

- `lsd` – це заміна стандартної команди `ls`, що дозволяє кольоровий вивід, значки для файлів та тек та інше, докладніше – <https://github.com/Peltoche/lsd>.
  
  ```sh
  brew install lsd
  ```

  Додамо втулок до Oh My Zsh

  ```sh
  git clone https://github.com/yuhonas/zsh-aliases-lsd.git \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-aliases-lsd
  ```

- `zsh-syntax-highlighting` – надає підсвітку синтаксису.

  Встановлення

  ```sh
  git clone https://github.com/zsh-users/zsh-syntax-highlighting.git \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
  ```

- `zsh-autosuggestions` – дає підказки під час вводу команд, що значно спрощує їх ввід.

  ```sh
  git clone https://github.com/zsh-users/zsh-autosuggestions \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
  ```

Ви завжди можете додати потрібні вам втулки, перелік яких є у <https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins>, або ж ви можете пошукати сторонні втулки та додати їх.

## Starship

Налаштуємо запрошення командного рядка. Докладні інструкції див <https://spaceship-prompt.sh/getting-started/#installing>

Клонуємо код до теки з власними темами

```sh
git clone https://github.com/spaceship-prompt/spaceship-prompt.git \
${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/themes/spaceship-prompt --depth=1
```

Створюємо символічне посилання

```sh
ln -s "$ZSH_CUSTOM/themes/spaceship-prompt/spaceship.zsh-theme" \
"$ZSH_CUSTOM/themes/spaceship.zsh-theme"
```

До `~/.zshrc` додаємо 

```sh
ZSH_THEME="spaceship"
SPACESHIP_DIR_TRUNC_REPO=false
```

## Шрифт

Встановимо шрифт

```sh
brew tap homebrew/cask-fonts
brew install font-hack-nerd-font
```

та додамо його в налаштуваннях термінала

![Hack Regular Nerd Font](/images/2023/02/hack-regular-nerd-font.png)

Тепер маємо такий вигляд запрошення командного рядка в терміналі, який з першого погляду дасть вам знати в якій гілці репозиторію ви зараз та чи є там зміни, які треба синхронізувати з віддаленим репо. 

![командний рядок в терміналі](/images/2023/02/terminal-spaceship-prompt.png)

## Власний профіль термінала

Ви можете обрати один з запропонованих профілів вигляду термінала – <kbd>⌘</kbd>+<kbd>I</kbd>, я надаю перевагу профілю – Toy Chest. Отримати його можна за посиланням – <https://github.com/JacksonGariety/toy-chest-theme>.

Завантажимо собі файл з налаштуваннями для термінала – <https://github.com/JacksonGariety/toy-chest-theme/blob/master/themes/Terminal/ToyChest.terminal> та імпортуємо його в термінал та зробимо його стандартним для всіх нових сесій.

![terminal ToyChest](/images/2023/02/terminal-toychest.png)

Тепер я маю термінал, який відповідає моїм потребам.

----

PS. Маленький додатковий штрих, щоб мати підсвічування синтаксису у **vim**, відкриємо (створимо) файл налаштувань `~/.vimrc`

```sh
vim ~/.vimrc
```

та додамо до нього наступне

```sh
filetype plugin indent on
syntax on
```

На виході маємо підсвічування синтаксису у **vim**.
![vim color](/images/2023/02/color-vim.png)