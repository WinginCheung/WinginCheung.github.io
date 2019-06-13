---
layout:     post
title:      Manage Project with Google Repo -- Example
subtitle:   用Google Repo管理多仓库项目之范例
date:       2019-06-13
author:     Wingin Cheung
header-img: img/post-google.jpg
catalog: true
mermaid: true
tags:
    - repo
---

# 用Google Repo管理多仓库项目之范例

## 1、远程git服务器

我们假设有一个远程git服务器，用于存储git仓库：

- 服务器名：`gitserver`

- IP地址：`192.168.1.200`

- 账户：`git`

- 现有仓库列表：

    ```shell
    git@gitserver:～/repository$ pwd
    /home/git/repository
    git@gitserver:～/repository$ tree -d
    .
    |-- apps
    |   |-- chromium.git
    |   |-- tcpdump.git
    |   |-- ping.git
    |-- hardware
        |-- broadcom
        |   |-- gps.git
        |   |-- nfc.git
        |   |-- wlan.git
        |-- akm.git
        |-- marvell.git
    ```
    
- 现有仓库tag信息：

    ```shell
    git@gitserver:～/repository$ find $(pwd) -name "*.git" | xargs -I {} \
    > sh -c "cd {}; echo {}\":\"; git tag"
    /home/git/repository/apps/chromium.git:
    v1.0
    v2.0
    /home/git/repository/apps/tcpdump.git:
    v0.1
    v0.2
    v1.0
    v1.1
    /home/git/repository/apps/ping.git:
    v1.0
    /home/git/repository/hardware/broadcom/gps.git:
    v1.0
    /home/git/repository/hardware/broadcom/nfc.git:
    v1.0
    v1.1
    /home/git/repository/hardware/broadcom/wlan.git:
    v1.0
    v1.1
    /home/git/repository/hardware/akm.git:
    v1.0
    /home/git/repository/hardware/marvell.git:
    v1.0
    ```
    
- 现有仓库branch信息：

    ```shell
    git@gitserver:～/repository$ find $(pwd) -name "*.git" | xargs -I {} \
    > sh -c "cd {}; echo {}\":\"; git branch"
    /home/git/repository/apps/chromium.git:
    * master
    /home/git/repository/apps/tcpdump.git:
    * master
      plat-v1.0
    /home/git/repository/apps/ping.git:
    * master
    /home/git/repository/hardware/broadcom/gps.git:
    * master
    /home/git/repository/hardware/broadcom/nfc.git:
    * master
    /home/git/repository/hardware/broadcom/wlan.git:
    * master
    /home/git/repository/hardware/akm.git:
    * master
    /home/git/repository/hardware/marvell.git:
    * master
    ```

    

## 2、客户端

为了显示客户端项目仓库的tag信息，我们准备一个名为ls_tag.sh的脚本用于显示指定目录下的git项目目录并打印相关tag信息：

```shell
#!/bin/sh

# file name: ls_tag.sh
# how to use: ls_tag.sh <path>

ls_tag(){
    for dir in `ls $1`; do \
        if [ -d "$1/${dir}" ]; then \
            if [ -d "$1/${dir}/.git" ]; then \
                echo $1/$dir":"
                cd $1/$dir
                git tag
            else \
                ls_tag $1/$dir
            fi     
        fi  
    done
}

if [ "$1"x == ""x ]; then \
    echo "Usage: " $0 " <path>"
else \
    ls_tag $1
fi
```

我们假定集成所有git仓库管理的客户端信息如下：

+ 客户端名称：`workPC`

+ IP地址：`192.168.1.101`

+ 用户名称：`workusr`

+ 期望的仓库目录：

    ```shell
    workusr@workPC:~/tag_repo$ pwd
    /home/workusr/tag_repo
    workusr@workPC:~/tag_repo$ tree -d
    .
    |-- external
    |   |-- apps
    |   |   |-- chromium
    |   |-- tcpdump
    |   |-- ping
    |-- drivers
    |   |-- gps
    |   |-- wlan
    |-- others
        |-- akm
        |-- marvell
        |-- nfc
    ```

+ 期望的仓库版本信息：

    ```shell
    /home/workusr/tag_repo/external/apps/chromium:
    tag -- v2.0
    /home/workusr/tag_repo/external/tcpdump:
    branch -- plat-v1.0
    /home/workusr/tag_repo/external/ping:
    branch -- master
    /home/workusr/tag_repo/drivers/gps:
    tag -- v1.0
    /home/workusr/tag_repo/drivers/wlan:
    branch -- master
    /home/workusr/tag_repo/others/akm:
    branch -- master
    /home/workusr/tag_repo/others/marvell:
    branch -- master
    /home/workusr/tag_repo/others/nfc:
    branch -- master
    ```

## 3、创建manifest

为了达到使用repo管理，我们需要准备manifest仓库。

我们先在远处git客户端建立一个manifests.git仓库，用于存储manifest相关文件：

```shell
workusr@workPC:~/tag_repo$ ssh 192.168.1.200 -l git
git@192.168.1.200\'s password:
git@gitserver:~$ cd ~/repository
git@gitserver:~/repository$ git init --bare manifests.git
Initialized empty Git repository in /home/git/repository/manifests.git/
git@gitserver:~/repository$ exit
workusr@workPC:~/tag_repo$
```

我们在客户端中clone远程git服务器中的manifests.git仓库：

```shell
workusr@workPC:~$ git clone git@192.168.1.200:/home/git/repository/manifests.git
Cloning into 'manifests'...
warning: You appear to have cloned an empty repository.
workusr@workPC:~$
```

好了，我们现在开始编辑default.xml文件：

```shell
workusr@workPC:~$ cd manifests
workusr@workPC:~/manifests$ vi default.xml
```

在打开的vi文件编辑界面中，输入以下内容并保存：

```xml
<?xml version='v1.0' encoding='UTF-8'?>

<manifest>
    <!-- server configuration -->
    <remote  name="git-server"
             fetch="ssh://git@192.168.1.200:/home/git/repository" />
    <default revision="master"
             remote="git-server"
             sync-j="4" />
    
    <!-- project configuration -->
    <project name="apps/chrominum" path="external/apps/chromium" revision="refs/tags/v2.0" />
    <project name="apps/tcpdump" path="external/tcpdump" revision="plat-v1.0" />
    <project name="apps/ping" path="external/ping" />
    <project name="hardware/broadcom/gps" path="drivers/gps" revision="refs/tags/v1.0" />
    <project name="hardware/broadcom/wlan" path="drivers/wlan" />
    <project name="hardware/akm" path="others/akm" />
    <project name="hardware/marvell" path="others/marvell" />
    <project name="hardware/broadcom/nfc" path="others/nfc" />
</manifest>
```

好了，default.xml文件我们已经准备好了，我们将其加入版本管理并上传到远程git服务器中：

```shell
workusr@workPC:~/manifests$ git add default.xml
workusr@workPC:~/manifests$ git commit -m "add defaul.xml"
workusr@workPC:~/manifests$ git push origin master
```

## 4、初始化tag_repo

OK，一切准备就绪，我们开始初始化tag_repo目录：

```shell
workusr@workPC:~/tag_repo$ repo init -u git@192.168.1.200:/home/git/repository/manifest.git
workusr@workPC:~/tag_repo$ repo sync
```