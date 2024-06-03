---
layout: "post"
title: "Deploy a React App to GitHub Pages"
date: "2024-06-03 10:30:00 +0200"
categories: en
tags:
  - React
  - GitHub
  - Actions
  - CI/CD
comments: true
lang: en
ref: 2024-06-03-deploy-react-app-to-github-pages

code:
  - url_param: "${{ steps.deployment.outputs.page_url }}"
  - token: "${{ secrets.GITHUB_TOKEN }}"
---

Here, I will show you how to deploy a React app to GitHub Pages using GitHub Actions. GitHub Pages is a static site hosting service that takes HTML, CSS, and JavaScript files straight from a repository on GitHub and makes it accessible as a website. GitHub Actions is a CI/CD service that allows you to automate your workflow on publishing your static site to GitHub Pages.

## Create a React App

First, create a new React app using Create React App.

```bash
npx create-react-app react-app --template typescript
cd react-app
```

Your git repository should be initialized automatically.

![Create React App 1](/images/2024/06/npx-boilerplating-1.png)
![Create React App 2](/images/2024/06/npx-boilerplating-2.png)

## Create a GitHub Repository

Working in the folder with the newly created app, create a new repository on GitHub.

```bash
gh repo create
```

![Create GitHub Repository](/images/2024/06/gh-repo-create.png)

## Adjust repository settings

Go to the repository settings and enable GitHub Pages. Choose the source for **Build and deployment** — GitHub Actions.

![GitHub Pages Settings](/images/2024/06/gh-repo-pages-source.png)

Open the **Actions**/**General** menu, scroll down to the **Workflow permissions** section, and give Read and Write permissions to the workflow.

![GitHub Actions Permissions](/images/2024/06/gh-repo-actions.png)

## Add GitHub Actions Workflow

Create a new directory called `.github/workflows` in the root of your repository.

```bash
mkdir -p .github/workflows
```

Create a new file called `deploy.yml` in the `.github/workflows` directory.

Name your workflow.

```yaml
name: Deploy React App
```

This will allow you to distinguish this workflow from others.

Add the following content to the `deploy.yml` file.

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

This section describes the events that will trigger the workflow. In this case, the workflow will be triggered on push to the main branch, when a pull request is closed, and when the workflow is manually triggered.

Next, add jobs description to the workflow.

```yaml
…
jobs:
  build:
    runs-on: ubuntu-latest
…
```

Our job with the name `build` will run on the latest version of Ubuntu.

Add steps to the job.

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

The first step checks out the code from the repository. The second step sets up Node.js.

Add more steps to the job.

```yaml
…
      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build
…
```

The third step installs dependencies, and the fourth step builds the React app.

Add the final steps to the job.

```yaml
…
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          name: 'github-pages'
          path: build
…
```

Here we use the `upload-pages-artifact` action to upload the build directory as an artifact for further deployment.

Create deployment steps.

```yaml
…
  deploy:
    if: github.event.pull_request.merged == true || github.event_name == 'push' || github.event_name == 'workflow_dispatch'
    needs: build

    permissions:
      pages: write
      id-token: write
      contents: read

    environment:
      name: github-pages
      url: url: {{ page.code[0].url_param }}
…
```

The deployment job will run only if the pull request is merged or the workflow is triggered by a push event or fired manually. It needs the `build` job to be completed. The deployment job has permissions to write to the pages, write the id-token, and read the contents. The environment name is `github-pages`, and the URL is the output of the deployment step.

Add steps to the deployment job.

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

The first step sets up the Pages. The second step deploys the build directory to GitHub Pages using the `deploy-pages` action. We use the previously built artifact and the GitHub token for authentication of the deployment.

Commit the changes and push them to the repository.

```bash
git add .
git commit -m "Add GitHub Actions workflow"
git push
```

## Adjust `package.json`

Open the `package.json` file and add homepage property. You can set it to `.` or provide the URL of your GitHub Pages (\"homepage\": \"<https://andygol.github.io/react-app/>\").

```diff
{
  "name": "react-app",
  "version": "0.1.0",
  "private": true,
+  "homepage": ".",
  "dependencies": {
    …
```

## Deploy the React App

Any edits you make to the `main` branch will trigger the GitHub Actions workflow. The workflow will build the React app and deploy it to GitHub Pages.

![Deployed React App](/images/2024/06/deployed-react-app.png)

<https://andygol.co.ua/react-app/>

That's it! You have successfully deployed a React app to GitHub Pages using GitHub Actions.

PS. If you want to see the full workflow file, check out the [GitHub repository](https://github.com/Andygol/react-app/blob/main/.github/workflows/deploy.yml).