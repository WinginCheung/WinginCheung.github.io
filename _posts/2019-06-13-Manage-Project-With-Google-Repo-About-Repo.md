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

***注：Repo安装及其使用不建议使用超级管理员权限执行！否则可能会出现一些异常情况，例如权限受限（Permission denied）：***

```shell
$ repo init -u git@192.168.1.200:/home/git/repository/manifests.git
Get https://mirrors.tuna.tsinghua.edu.cn/git/git-repo

...

type commit
tag v1.13.3
tagger Mike Frysinger <vapier@google.com> 1559612018 -0400

repo 1.13.3

gpg: WARNING: unsafe owership on homedir '/home/workusr/.repoconfig/gnupg'
gpg: failed to create temporary file '/home/workusr/.repoconfig/gnupg/.#lk0x00005647f9f17a70.server.25250': Permission denied
gpg: keyblock resource '/home/workusr/.repoconfig/gnupg/pubring.kbx': Permission denied
gpg: Signature made Tue 04 Jun 2019 09:33:39 AM CST
gpg:                using DSA key 8BB9AD73E8E6153AF0F9A4416530D6R920F6C65
gpg:                issuer "repo@android.kernel.org"
gpg: Can't signature: No public key
```



## 3、Repo更新

### 3.1 修改REPO_URL

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

或者在执行`repo init`时指定REPO_URL：

```shell
$ repo init -u <manifest-url> --repo-url=<repo-repository-url>
```

### 3.2 Repo仓库本地化

每次运行repo init命令时，repo将会尝试更新自己，但有时网络异常时，我们将无法连接到指定的`REPO_URL`服务器更新repo，造成无法创建仓库：

```shell
$ repo init -u <url>
fatal: Cannot get https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/clone.bundle
fatal: error [Error -2] Name or server not known
fatal: cloning the git-repo repository failed, will remove '.repo/repo'
```

为了解决此问题，我们可以将repo仓库备份到指定服务器，在指定服务器中执行以下操作：

```shell
$ git clone --mirror https://mirrors.tuna.tsinghua.edu.cn/git/git-repo
```

并参考`3.1 修改REPO_URL`将REPO_URL上述REPO_URL修改为指定服务器的repo仓库即可。

如需更新指定服务器中的repo仓库，可在指定服务器中的repo仓库下执行以下指令：

```shell
$ git fetch
```

