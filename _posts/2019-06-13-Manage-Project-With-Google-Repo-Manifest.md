---
layout:     post
title:      Manage Project with Google Repo -- Manifest
subtitle:   用Google Repo管理多仓库项目之Manifest
date:       2019-06-13
author:     Wingin Cheung
header-img: img/post-google.jpg
catalog: true
mermaid: true
tags:
    - repo
---

# 用Google Repo管理多仓库项目之Manifest

Repo管理的核心就在于Manifest，每个采用repo管理的复杂多仓库项目都需要一个对应的manifest仓库，用来存储所有仓库的配置信息。Repo也是读取此仓库的配置文件来进行管理操作。manifest的配置是xml定义的结构，主要有两个配置：

* 远程服务器信息
* 子项目详细配置信息

我们摘取[CyanogenMod/android/default.xml](https://github.com/CyanogenMod/android/blob/cm-14.1/default.xml)来解析一下manifest的配置信息：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>

    <remote  name="github"
             fetch=".."
             review="review.cyanogenmod.org" />

    <remote  name="private"
             fetch="ssh://git@github.com" />

    <remote  name="aosp"
             fetch="https://android.googlesource.com"
             review="android-review.googlesource.com"
             revision="refs/tags/android-7.1.1_r6" />

    <default revision="refs/heads/cm-14.1"
             remote="github"
             sync-c="true"
             sync-j="4" />

    <!-- AOSP Projects -->

    <project path="build" name="CyanogenMod/android_build" groups="pdk,tradefed">
        <copyfile src="core/root.mk" dest="Makefile" />
    </project>
    <project path="build/blueprint" name="platform/build/blueprint" groups="pdk,tradefed" remote="aosp" />
    <project path="build/kati" name="CyanogenMod/android_build_kati" groups="pdk,tradefed" />
    <project path="build/soong" name="platform/build/soong" groups="pdk,tradefed" remote="aosp" >
        <linkfile src="root.bp" dest="Android.bp" />
        <linkfile src="bootstrap.bash" dest="bootstrap.bash" />
    </project>

    ...

    <project path="external/chromium-webview" name="platform/external/chromium-webview" groups="pdk" clone-depth="1" remote="aosp" />

    ...

    <project path="packages/apps/TV" name="CyanogenMod/android_packages_apps_TV" />

    <include name="snippets/cm.xml" />

</manifest>
```

## 1、xml声明

只能放置在xml文件的最前面：

```xml
<?xml version="1.0" encoding="UTF-8"?>
```

## 2、manifest元素

最顶层xml元素：

```xml
<manifest>
 ...
</manifest>
```

## 3、remote元素

设置远程git服务器的属性：

```xml
<remote  name="github"
         fetch=".."
         review="review.cyanogenmod.org" />

<remote  name="private"
         fetch="ssh://git@github.com" />

<remote  name="aosp"
         fetch="https://android.googlesource.com"
         review="android-review.googlesource.com"
         revision="refs/tags/android-7.1.1_r6" />
```

### 3.1 name

指定远程服务器的名称，直接用于git fetch，git remote等操作，例如，设置远程git服务器为`github`：

```xml
name="github"
```

### 3.2 alias

远程git服务器的别名，若有指定，则***覆盖上边name的设定***。***在一个manifest中，name值不能重名，但是alias可以重名***；

### 3.3 fetch

git URL，是相关project的git URL前缀；如上边的：

- 指定仓库下载的位置为本manifest工程的上一级目录：

- ```xml
    fetch=".."
    ```

- 指定远程服务器地址为`ssh://git@github.com`：

- ```xml
    fetch="ssh://git@github.com"
    ```

- 指定远程服务器地址为`https://android.googlesource.com`：

- ```xml
    fetch="https://android.googlesource.com"
    ```

### 3.4 review

指定Gerrit服务器，用于`repo upload`操作，若未指定，则`repo upload`操作无效。如设定gerrit代码审核服务器的位置为`review.cyanogenmod.org`：

```xml
review="review.cyanogenmod.org"
```

### 3.5 revision

指定默认git分支。如设定默认git分支为`refs/tags/android-7.1.1_r6`：

```xml
revision="refs/tags/android-7.1.1_r6"
```

## 4、default元素

指定默认远程git服务器的属性：

```xml
<default revision="refs/heads/cm-14.1"
         remote="github"
         sync-c="true"
         sync-j="4" />
```

### 4.1 revision

指定默认git分支，如：

- `master`：

- ```xml
  revision="master"
  ```

- `refs/heads/cm-14.1`：

- ```xml
  revision="refs/heads/cm-14.1"
  ```

### 4.2 remote

指定默认远程服务器为上边remote元素中声明的服务器，如`github`：

```xml
remote="github"
```

### 4.3 sync-c

若设置为true，则只同步指定的分支，即上边的default元素中的revision指定的分支，而不是所有的ref内容。

### 4.4 sync-s

若设置为true，则会同步git的子项目。

### 4.5 sync-j

指定`repo sync`的默认并行数目，如范例default元素中指定其值为4。

## 5、xml注释

```xml
<!-- AOSP Projects -->
```

## 6、project元素

具体子项目信息：

```xml
<project path="build" name="CyanogenMod/android_build" groups="pdk,tradefed">
    <copyfile src="core/root.mk" dest="Makefile" />
</project>
<project path="build/blueprint" name="platform/build/blueprint" groups="pdk,tradefed" remote="aosp" />
<project path="build/kati" name="CyanogenMod/android_build_kati" groups="pdk,tradefed" />
<project path="build/soong" name="platform/build/soong" groups="pdk,tradefed" remote="aosp" >
    <linkfile src="root.bp" dest="Android.bp" />
    <linkfile src="bootstrap.bash" dest="bootstrap.bash" />
</project>

...

<project path="external/chromium-webview" name="platform/external/chromium-webview" groups="pdk" clone-depth="1" remote="aosp" />
```

### 6.1 工程子项目节点声明

```xml
<project ... />
```

若中间包含其它节点，如copyflie节点，则为：

```xml
<project ...
    <copyfile ... />
</project>
```

### 6.2 path

可选的路径。指定git clone出来的代码存放在本地的子目录。如果没有指定，则以name作为子目录名。如下载工程代码到本地工程的build/buleprint目录：

```xml
path="build/blueprint"
```

### 6.3 name

唯一的名字标识project，同时也用于生成git仓库的URL。格式为`$(remote_fetch)/$(project_name).git`。

如上边第一个project的name值为`CyanogenMod/android_build`，未指定remote信息，则该工程代码的git仓库为默认服务器`github`的`../CyanogenMod/android_build.git`；

### 6.4 groups

列出project所属的组，以空格或者逗号分隔多个组名。

- 所有的project都自动属于"all"组；

- 每一个project自动属于name:'name' 和path:'path'组。例如上边的第三个project指定groups为pdk组和tradefed组；若去掉groups字段，它自动属于default, name:CyanogenMod/android_build_kati和 path:build/kati组；

- 如果一个project属于notdefault组，则repo sync时不会下载

### 6.5 clone-depth

指定git clone的深度，为1表示只clone最近一次commit；

### 6.6 remote

指定远程服务器，如第二个project指定远程服务器为`remote="aosp"`，即其地址为`https://android.googlesource.com`：

### 6.7 copyfile/linkfile节点

- src：源文件；
- dest：目标文件；

### 6.8 revision

指定需要获取的git分支，可以是master, refs/heads/master, tag或者SHA-1值；

### 6.9 sync_c

如果设置为true，则只同步指定的分支(revision 属性指定)，而不是所有的ref内容；

### 6.10 sync_s

如果设置为true，则会同步git的子项目；

### 6.11 upstream

在哪个git分支可以找到一个SHA1。用于同步revision锁定的manifest(-c 模式)。该模式可以避免同步整个ref空间；

### 6.12 annotation

可以有多个annotation，格式为name-value pair。在repo forall 命令中这些值会导入到环境变量中；

### 6.13 remove-project

从内部的manifest表中删除指定的project。经常用于本地的manifest文件，用户可以替换一个project的定义；

## 7、include

 通过name属性可以引入另外一个manifest文件(路径相对与manifest repository's root)，如范例中最后引入了本文件同目录下的snippets文件夹中名为cm.xml的xml文件。