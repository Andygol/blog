---
layout: post
title: Updating zsh in Ventura 13.2 on a MacBook pro
date: '2023-02-03 12:00:00 +0200'
categories:
  - en
tags:
  - macOS
  - zsh
comments: true
lang: en
ref: m1-zsh-ventura-13.2
---
The other day, I installed the next macOS Ventura update — version 13.2. And the question arose before me, should I use the built-in shell, or should I update it?

A quick web search yielded no answers, so I decided to compare the versions.

```sh
zsh --version
```

**zsh 5.8.1 (×86_64-apple-darwin22.0)**&nbsp;— the typical version supplied by Apple.

Checking which version is in Homebrew

```sh
brew info zsh
```

![Azsh default version in Ventura 13.2](/images/2023/02/zsh-m1-ventura-13.2.png)
**zsh: stable 5.9 (bottled), HEAD**&nbsp;— version that can be obtained via Homebrew.

At first glance, we can see that Homebrew has a newer version. However, this is what catches my eye — <kbd>×86_64</kbd>, which suggests that this is a version for Intel, not for Apple Silicon. 😕

## The decision is to update!

```sh
brew install zsh
```

Restart shell

```sh
source ~/.zshrc
```

and check the version of the installed shell

```sh
zsh --version
```

![Azsh updated via brew](/images/2023/02/zsh-m1-brew.png)

Let's compare

> zsh 5.8.1 (***×86_64***-apple-darwin22.0) — Intel (Ventura 13.2)  
> zsh 5.9 (***arm-***apple-darwin22.1.0) — Apple Silicon (Homebrew)

The updated version of zsh 5.9 is the version specifically for Apple Silicon 🚀, it is strange that the version of macOS created for M1/M2 contains an assembly for Intel.

If you have an M1/M2 Apple Silicon — I advise you to install the version of zsh compiled specifically for your processor.
