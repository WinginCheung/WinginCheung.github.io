---
layout:     post
title:      Git Daemon
subtitle:   Git Daemon在Ubuntu18.04中的安装使用
date:       2019-07-11
author:     Wingin Cheung
header-img: img/git-server.jpg
catalog: true
mermaid: true
tags:
    - git


---

# Git Daemon在Ubuntu18.04中的安装使用

## 1、创建git账户

创建名为git的账户：

```shell
$ sudo useradd -m git
```

给git账户设置密码：

```shell
$ sudo passwd git
```

## 2、安装git-daemon

更新系统：

```shell
$ sudo apt update
$ sudo apt upgrade
```

安装runit-systemd和git-daemon：

```shell
$ sudo apt install runit-systemd git-daemon
```

## 3、配置git-daemon

修改配置文件/etc/sv/git-daemon/run，并指定repository根目录，例如`/home/git/repository`：

> \#!/bin/sh
>
> exec 2>&1
>
> echo 'git-daemon starting.'
>
> exec chest -ugitdaemon:git \
>
> ​    "$(git --exec-path)"/git-daemon —verbose --enable=receive-pack --export-all --reuseaddr \
>
> ​    \--base-path=/home/git/repository

## 4、创建/var/lib/supervise/git-daemon目录

```shell
$ sudo mkdir -p /var/lib/supervise/git-daemon
```

***注：若不手动创建该目录，启动git-daemon服务可能会失败：***

```shell
$ sudo sv restart git-daemon
warning: git-daemon: unable to open supervise/ok: file dose not exist
```

## 5、启动git-daemon服务

```shell
$ sudo sv restart git-daemon
ok: run: git-daemon: (pid 27854) 0s
```

## 6、使用git-daemon服务clone仓库

假如我们需要clone指定的repository根目录（`/home/git/repository`）下的仓库`git_test/test.git`：

```shell
$ git clone git://<ip-to-mirror>/git_test/test.git
```

## 7、错误集锦

### 7.1 git-daemon启动失败

```shell
$ sudo sv restart git-daemon
timeout: down: git-daemon: 0s, normally up, want up
```

请检查以下配置：

（1）/etc/sv/git-daemon/run中设置的repository根目录是否存在；

（2）/etc/sv/git-daemon/run中-ugitdaemon后是否添加git账户用户，如`:git`；

（3）若是（2）中的git账户未添加，repository根目录下的目录other用户是否有rx权限，若无，可使用以下修改：

```shell
$ sudo find <repo-base-path> -d | xargs -I {} sudo chmod o+r {}
$ sudo find <repo-base-path> -d | xargs -I {} sudo chmod o+x {}
```

### 7.2 git clone仓库失败

```shell 
$ git clone git://<ip-to-mirror>/git_test/test.git
Clone into 'test'...
fatal: remote error: access denied or repository not exported: /git_test/test.git
```

请检查一下配置：

（1）/etc/sv/git-daemon/run中设置的repository根目录是否存在；

（2）\<repo-base-path\>/\<repo-path\>是否存在；

（3）/etc/sv/git-daemon/run中-ugitdaemon后是否添加git账户用户，如`:git`；

（4）若是（3）中的git账户未添加，仓库目录other用户是否有rx权限，若无，可使用以下修改：

```shell
$ sudo find <repo-base-path>/<repo-path> -d | xargs -I {} sudo chmod o+r {}
$ sudo find <repo-base-path><repo-path> -d | xargs -I {} sudo chmod o+x {}
```

### 7.3 git push失败

```shell
$ git push
fatal: remote error: access denied or repository not exported: /git_test/test.git
```

请检查一下配置：

（1）/etc/sv/git-daemon/run中设置的repository根目录是否存在；

（2）\<repo-base-path\>/\<repo-path\>是否存在；

（3）/etc/sv/git-daemon/run中-ugitdaemon后是否添加git账户用户，如`:git`；

（4）/etc/sv/git-daemon/run中是否添加`--enable=receive-pack`字段；