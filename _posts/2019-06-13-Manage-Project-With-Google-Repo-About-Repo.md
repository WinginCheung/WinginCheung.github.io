---
layout:     post
title:      Manage Project with Google Repo -- About Repo
subtitle:   用Google Repo管理多仓库项目之Repo介绍
date:       2019-06-13
author:     Wingin Cheung
header-img: img/post-google.jpg
catalog: true
mermaid: true
tags:
    - repo
---

# 用Google Repo管理多仓库项目之Repo介绍

来自[清华大学开源软件镜像站 — Git Repo镜像使用帮助](https://mirrors4.tuna.tsinghua.edu.cn/help/git-repo/)



## 1、关于Repo

> Repo is a tool that we built on top of Git. Repo helps us manage the many Git repositories, does the uploads to our revision control system, and automates parts of the Android development workflow. Repo is not meant to replace Git, only to make it easier to work with Git in the context of Android. The repo command is an executable Python script that you can put anywhere in your path.

Repo是对Git构成补充的多代码库管理工具，简单的说就是在Git基础上使用Python开发的一系列的脚步命令工具。Android Open Source Project（AOSP）就是使用它来管理的。

## 2、Repo安装

Repo托管于googlesource.com。但目前相关网站被墙，无法连接，我们可从清华大学开源镜像站获取相关代码：

```shell
$ curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo
$ chmod +x repo
```

## 3、Repo更新

repo的运行过程中会尝试访问官方的git源更新自己，如果想使用tuna的镜像源进行更新，可以将如下内容写入到`~/.bashrc`或者`/etc/bash.bashrc`里：

```shell
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
```

重启后生效。

我们也可以直接修改Repo源码中的REPO_URL字段：

```shell
...
if not REPO_URL:
    REPO_URL = 'https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
REPO_REV = 'stable'
...
```
