---
layout: "post"
title: "Розгортаємо застосунок React на GitHub Pages"
date: "2024-06-03 10:30:00 +0200"
categories: uk
tags:
  - React
  - GitHub
  - Actions
  - CI/CD
comments: true
lang: uk
ref: 2024-06-03-deploy-react-app-to-github-pages

code:
  - url_param: "${{ steps.deployment.outputs.page_url }}"
  - token: "${{ secrets.GITHUB_TOKEN }}"
---

Тут я покажу вам, як розгорнути застосунок React на GitHub Pages за допомогою GitHub Actions. GitHub Pages — це сервіс хостингу статичних сайтів, який бере HTML, CSS та JavaScript файли безпосередньо з репозиторію на GitHub та робить їх доступними у вигляді вебсайту. GitHub Actions — це сервіс CI/CD, що дозволяє автоматизувати робочий процес публікації вашого статичного сайту на GitHub Pages.

## Створення застосунку React

Спочатку створіть новий застосунок React за допомогою Create React App.

```bash
npx create-react-app react-app --template typescript
cd react-app
```

Ваш git-репозиторій має бути ініціалізований автоматично.

![Create React App 1](/images/2024/06/npx-boilerplating-1.png)
![Create React App 2](/images/2024/06/npx-boilerplating-2.png)

## Створення репозиторію на GitHub

Перебуваючи у теці з новоствореним застосунком та створіть новий репозиторій на GitHub.

```bash
gh repo create
```

![Create GitHub Repository](/images/2024/06/gh-repo-create.png)

## Налаштування репозиторію

Перейдіть до налаштувань репозиторію та увімкніть GitHub Pages. Оберіть джерело для **Build and deployment** — GitHub Actions.

![GitHub Pages Settings](/images/2024/06/gh-repo-pages-source.png)

Відкрийте меню **Actions**/**General**, прокрутіть до розділу **Workflow permissions** та надайте дозволи на читання та запис для робочого процесу GitHub Actions.

![GitHub Actions Permissions](/images/2024/06/gh-repo-actions.png)

## Додавання робочого процесу GitHub Actions

Створіть нову теку `.github/workflows` у корені вашого репозиторію.

```bash
mkdir -p .github/workflows
```

Створіть новий файл з назвою (наприклад) `deploy.yml` у теці `.github/workflows`.

Додайте назву вашого робочого процесу.

```yaml
name: Deploy React App
```

Це дозволить вам вирізняти цей робочий процес серед інших.

Додайте наступний вміст до файлу `deploy.yml`.

```yaml
…

on:
    push:
        branches: [main]
    pull_request:
        types:
            - closed
        branches: [main]
    workflow_dispatch:
…
```

Цей розділ описує події, що запускають робочий процес. У цьому випадку робочий процес буде запущений при безпосередньому внесенні змін в гілку main за допомогою `git push`, при виконанні pull request та при ручному запуску робочого процесу.

Додайте опис завдань до робочого процесу.

```yaml
…
jobs:
  build:
    runs-on: ubuntu-latest
…
```

Наше завдання з ім'ям `build` буде виконуватись на останній версії Ubuntu.

Додайте кроки до завдання.

```yaml
…
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
…
```

Перший крок витягує код з репозиторію. Другий крок налаштовує Node.js.

Додайте більше кроків до завдання.

```yaml
…
      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build
…
```

Третій крок встановлює залежності, а четвертий крок збирає застосунок React.

Додайте останні кроки до завдання.

```yaml
…
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          name: 'github-pages'
          path: build
…
```

Тут ми використовуємо дію `upload-pages-artifact` для завантаження теки збірки як артефакту для подальшого розгортання.

Створіть кроки розгортання.

```yaml

  deploy:
    if: github.event.pull_request.merged == true || github.event_name == 'push' || github.event_name == 'workflow_dispatch'
    needs: build

    permissions:
      pages: write
      id-token: write
      contents: read

    environment:
      name: github-pages
      url: {{ page.code[0].url_param }}

```

Завдання розгортання буде виконуватись тільки якщо pull request обʼєднано з основною гілкою або робочий процес запускається через `git push` або вручну. Воно потребує завершення завдання `build`. Завдання розгортання має дозволи на запис до сторінок, запис токена id та читання вмісту. Ім'я середовища — `github-pages`, а URL — покаже вам адресу після завершення кроку розгортання.

Додайте кроки до завдання розгортання.

```yaml
…
    runs-on: ubuntu-latest
    steps:
      - name: Setup Pages
        uses: actions/configure-pages@v5

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
        with:
          token: {{ page.code[1].token }}
          artifact_name: 'github-pages'
```

Перший крок налаштовує Pages. Другий крок розгортає теку збірки на GitHub Pages за допомогою `deploy-pages`. Ми використовуємо артефакт попередньо створений на етапі збірки (`build`) та GitHub token для автентифікації розгортання.

Тепер зробіть commit змін та надішліть їх у репозиторій.

```bash
git add .
git commit -m "Add GitHub Actions workflow"
git push
```

## Налаштування `package.json`

Відкрийте файл `package.json` і додайте параметр `homepage`. Ви можете встановити його значення як `.` або надати URL вашого репо для GitHub Pages (\"homepage\": \"<https://andygol.github.io/react-app/>\").

```diff
{
  "name": "react-app",
  "version": "0.1.0",
  "private": true,
+  "homepage": ".",
  "dependencies": {
    …
```

## Розгортання застосунку React

Будь-які зміни, які ви вносите до гілки `main`, запустять робочий процес GitHub Actions. Робочий процес збере застосунок React і розгорне його на GitHub Pages.

![Deployed React App](/images/2024/06/deployed-react-app.png)

<https://andygol.co.ua/react-app/>

Ось і все! Ви успішно розгорнули застосунок React на GitHub Pages за допомогою GitHub Actions.

PS. Якщо ви хочете побачити повний файл робочого процесу, перегляньте [репозиторій на GitHub](https://github.com/Andygol/react-app/blob/main/.github/workflows/deploy.yml).