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
    - example
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
    |   |       |-- chromium.c
    |   |       |-- ...
    |   |-- tcpdump
    |   |   |-- tcpdump.c
    |   |   |-- ...
    |   |-- ping
    |       |-- ping.c
    |       |-- ...
    |-- drivers
    |   |-- gps
    |   |   |-- gps.c
    |   |   |-- ...
    |   |-- wlan
    |       |-- wlan.c
    |       |-- ...
    |-- others
        |-- akm
        |   |-- akm.c
        |   |-- ...
        |-- marvell
        |   |-- marvell.c
        |   |-- ...
        |-- nfc
            |-- nfc.c
            |-- ...
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
```

我们在客户端中clone远程git服务器中的manifests.git仓库：

```shell
workusr@workPC:~$ git clone git@192.168.1.200:/home/git/repository/manifests.git
Cloning into 'manifests'...
warning: You appear to have cloned an empty repository.
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
workusr@workPC:~/manifests$ mkdir -p ~/tag_repo
workusr@workPC:~/manifests$ cd ~/tag_repo
workusr@workPC:~/tag_repo$ repo init -u git@192.168.1.200:/home/git/repository/manifest.git
...
workusr@workPC:~/tag_repo$ repo sync
...
```

## 5、创建分支

`repo sync`完成后，所有的仓库均处于no branch状态，不能进行仓库操作

```shell
workusr@workPC:~/tag_repo$ repo branch
    (no branches)
```

我们需要对指定项目初始化分支，例如对所有项目：

```shell
workusr@workPC:~/tag_repo$ repo start newbranch --all
```

也可以只对ping子项目：

```shell
workusr@workPC:~/tag_repo$ repo start pingbranch external/ping
```

初始化分支后，我们可以使用`repo branch`查看：

```shell
workusr@workPC:~/tag_repo$ repo branch
...
*  newbranch                | in all projects
*  pingbranch               | in external/ping
```

## 6、项目文件修改及其提交

初始化分支后，我们接下来的操作与git相同。

现在我们来修改文件并提交相关修改，以下以子项目ping中的ping.c修改为例。

我们先查看一下创建分支后我们所有项目的状态：

```shell
workusr@workPC:~/tag_repo$ repo status
...
project external/apps/chromium/            branch newbranch
project external/tcpdump/                  branch newbranch
project external/ping/                     branch pingbranch
project drivers/gps/                       branch newbranch
project drivers/wlan/                      branch newbranch
project others/akm/                        branch newbranch
project others/marvell/                    branch newbranch
project others/nfc/                        branch newbranch
```

下面我们进行文件修改操作，例如在ping.c最后添加一句“hello world“，然后再查看一下项目状态：

```shell
workusr@workPC:~/tag_repo$ cd external/ping
workusr@workPC:~/tag_repo/external/ping$ echo "hello world." >> ping.c
workusr@workPC:~/tag_repo/external/ping$ repo status
...
project external/apps/chromium/            branch newbranch
project external/tcpdump/                  branch newbranch
project external/ping/                     branch pingbranch
 -m     ping.c
project drivers/gps/                       branch newbranch
project drivers/wlan/                      branch newbranch
project others/akm/                        branch newbranch
project others/marvell/                    branch newbranch
project others/nfc/                        branch newbranch
```

可以看到ping.c被标记为"-m"，即被修改状态。

我们使用git命令将其修改提交：

```shell
workusr@workPC:~/tag_repo/external/ping$ git add ping.c
workusr@workPC:~/tag_repo/external/ping$ repo status
...
project external/apps/chromium/            branch newbranch
project external/tcpdump/                  branch newbranch
project external/ping/                     branch pingbranch
 M+     ping.c
project drivers/gps/                       branch newbranch
project drivers/wlan/                      branch newbranch
project others/akm/                        branch newbranch
project others/marvell/                    branch newbranch
project others/nfc/                        branch newbranch
workusr@workPC:~/tag_repo/external/ping$ git commit -m "modify test"
...
workusr@workPC:~/tag_repo/external/ping$ repo status
...
project external/apps/chromium/            branch newbranch
project external/tcpdump/                  branch newbranch
project external/ping/                     branch pingbranch
project drivers/gps/                       branch newbranch
project drivers/wlan/                      branch newbranch
project others/akm/                        branch newbranch
project others/marvell/                    branch newbranch
project others/nfc/                        branch newbranch
```

到此，我们已经将其修改提交到本地的版本库了。

我们还可以继续将本次修改提交到远程仓库：

```shell
workusr@workPC:~/tag_repo/external/ping$ git push --all
```

我们可以在远程仓库执行`git branch`，确实是否有新提交的名为`pingbranch`的分支；可以执行`git show pingbranch`查看详细的提交信息。

在子项目下，我们可以直接使用git相关指令来控制项目版本，也可以在git命令前添加`repo forall -c`来控制项目版本，两者效果相同。