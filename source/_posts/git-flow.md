---
title: Git Flow
date: 2021-06-10 11:21:00
categories: 
	- 前端工程化
tags: 
	- Git
	- 入门教程
---

## 安装 git-flow

```bash
$ git flow init
Initialized empty Git repository in /Users/tobi/acme-website/.git/
Branch name for production releases: [master]
Branch name for "next release" development: [develop]

How to name your supporting branch prefixes?
Feature branches? [feature/]
Release branches? [release/]
Hotfix branches? [hotfix/]
```

## 新功能开发

### 1.新建一个功能分支

```bash
$ git flow feature start rss-feed
Switched to a new branch 'feature/rss-feed'
```

### 2.完成一个功能

```bash
$ git flow feature finish rss-feed
```

功能合并到 dev 分支并切换到 dev 分支（在此之前 dev 分支的代码要拉到最新）

## 管理 releases

### 1.创建 releases

```bash
$ npm version patch
$ git flow release start 1.1.5

Switched to a new branch 'release/1.1.5'
```

### 2.完成 releases (在此之前 master 代码要拉到最新)

```bash
$ git flow release finish 1.1.5
```

## 推送本地代码（dev 和 master）和 tag

```bash
$ git push --tags
```

更详细讲解: [链接](https://www.git-tower.com/learn/git/ebook/cn/command-line/advanced-topics/git-flow/)
