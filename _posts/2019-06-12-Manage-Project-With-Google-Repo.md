---
layout:     post
title:      Manage Projects with Google Repo
subtitle:   用Repo管理工程代码
date:       2019-06-12
author:     Wingin Cheung
header-img: img/markdown-write.jpg
catalog: true
mermaid: true
tags:
    - repo
---

# 用Google Repo管理工程代码

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

## 4、Repo命令

Repo命令的基本格式如下：

```shell
$ repo <command> [options] 
```

可选元素显示在方括号 [ ] 中。例如，许多命令会用到项目列表 (project-list) 参数。项目列表可以是一个名称列表，也可以是一个本地源代码目录的路径列表：

```shell
$ repo sync [project0 project1 ... projectn]
$ repo sync [/path/to/project0 ... /path/to/projectn]
```

### 4.1 repo help

安装 Repo 后，您可以通过运行以下命令找到最新文档（开头是包含所有命令的摘要）：

```shell
$ repo help
```

也可以通过在 Repo 树中运行以下命令来获取有关某个命令的信息：

```shell
$ repo help <command>
```

例如，要获取`repo init`的帮助文档，我们可以使用以下指令：

```shell
$ repo help init
```

### 4.2 repo init

`repo init`的基本格式为：

```shell
$ repo init -u <url> [options]
```

该指令会在当前目录中安装Repo，创建一个`.repo`目录，其中包含Repo源代码和指向的工程清单文件的git代码库。该`.repo`目录中，还包含一个指向`.repo/mainfests/`目录中所选清单的符号链接`manifest.xml`。

选项：

* -u：指定要从中检索清单代码库的网址。若是AOSP，则可从`https://android.googlesource.com/platform/manifest` 中找到通用清单；
* -m：选择代码库中的一个清单文件。如果未选择任何清单名称，则会默认选择 `default.xml`；
* -b：指定修订版本，即特定的清单分支；

### 4.3 repo sync

`repo sync`的基本格式为：

```shell
$ repo sync [project-list]
```

该操作会下载新的更改并更新本地环境中的工作文件。若未指定[project-list]，则该操作会同步所有项目的文件。

运行`repo sync`后，可能会出现以下情况：

* 如果目标项目从未同步过，则 `repo sync` 相当于 `git clone`。远程代码库中的所有分支都会复制到本地项目目录中；

* 如果目标项目以前同步过，则 `repo sync` 相当于以下命令：

    ```shell
    $ git remote update
    $ git rebase origin/<branch>
    ```

    其中 `branch` 是本地项目目录中当前已检出的分支。如果本地分支没有在跟踪远程代码库中的分支，则项目不会发生任何同步。

* 如果 `Git rebase`操作导致合并冲突，请使用常规 Git 命令（例如 `git rebase --continue`）解决冲突。

`repo sync`运行成功后，指定项目中的代码即处于最新状态，已与远程代码库中的代码同步。

选项：

- `-d`：将指定项目切换回清单修订版本。如果项目当前属于某个主题分支，但临时需要清单修订版本，则此选项会有所帮助。
- `-s`：同步到当前清单中的 manifest-server 元素指定的一个已知良好版本。
- `-f`：即使某个项目同步失败，也继续同步其他项目。

### 4.4 repo upload

`repo upload`的基本格式为：

```shell
$ repo upload [project-list]
```

对于指定的项目，Repo 会将本地分支与最后一次 repo sync 时更新的远程分支进行比较。Repo 会提示选择一个或多个尚未上传以供审核的分支。

接下来，所选分支上的所有提交都会通过 HTTPS 连接传输到 Gerrit。您需要配置一个 HTTPS 密码以启用上传授权。要生成新的用户名/密码对以用于 HTTPS 传输。

当 Gerrit 通过其服务器接收对象数据时，它会将每项提交转变成一项更改，以便审核者可以针对特定提交给出意见。要将几项“检查点”提交合并为一项提交，请使用 `git rebase -i`，然后再运行 upload。

如果您在未使用任何参数的情况下运行 `repo upload`，则该操作会搜索所有项目中的更改以进行上传。

要在更改上传后对其进行修改，请使用 `git rebase -i` 或 `git commit --amend` 等工具更新您的本地提交。修改完成之后，请执行以下操作：

- 进行验证以确保更新后的分支是当前已检出的分支。

- 对于相应系列中的每项提交，请在方括号内输入 Gerrit 更改 ID：

    ```shell
    # Replacing from branch foo
    [ 3021 ] 35f2596c Refactor part of GetUploadableBranches to lookup one specific...
    [ 2829 ] ec18b4ba Update proto client to support patch set replacments
    # Insert change numbers in the brackets to add a new patch set.
    # To create a new change record, leave the brackets empty.
    ```

上传完成后，这些更改将拥有一个额外的补丁程序集。

如果您希望只上传当前已检出的 Git 分支，则可以使用标记 `--current-branch`（简称 `--cbr`）。

### 4.5 repo diff

`repo diff`的基本格式为：

```shell
$ repo diff [project-list]
```

使用 `git diff` 显示提交与工作树之间的明显更改。

### 4.6 repo download

`repo download`的基本格式为：

```shell
$ repo download target change
```

从审核系统中下载指定更改，并放在您项目的本地工作目录中供使用。

例如，要将名为"23823"的更改下载到您的 platform/build 目录，请运行以下命令：

```shell
$ repo download platform/build 23823
```

运行 `repo sync` 会删除使用 `repo download` 检索到的任何提交。或者，您可以使用 `git checkout m/master` 检出远程分支。

***注意：由于全球的所有服务器均存在复制延迟，因此某项更改出现在网络上（位于 [Gerrit](https://android-review.googlesource.com/) 中）的时间与所有用户可通过 `repo download` 找到此项更改的时间之间存在些许的镜像延迟。***

### 4.7 repo forall

`repo forall`的基本格式为：

```shell
$ repo forall [project-list] -c <command>
```

在每个项目中运行指定的 shell 命令。通过 `repo forall` 可使用下列额外的环境变量：

- `REPO_PROJECT` 设为了项目的唯一名称。
- `REPO_PATH` 是相对于客户端根目录的路径。
- `REPO_REMOTE` 是清单中远程系统的名称。
- `REPO_LREV` 是清单中修订版本的名称，已转换为本地跟踪分支。如果您需要将清单修订版本传递到某个本地运行的 Git 命令，则可使用此变量。
- `REPO_RREV` 是清单中修订版本的名称，与清单中显示的名称完全一致。

选项：

- `-c`：要运行的命令和参数。此命令会通过 `/bin/sh` 进行评估，它之后的任何参数都将作为 shell 位置参数传递。
- `-p`：在所指定命令的输出结果之前显示项目标头。这通过以下方式实现：将管道绑定到命令的 stdin、stdout 和 sterr 流，然后通过管道将所有输出结果传输到一个分页会话中显示的连续流中。
- `-v`：显示该命令向 stderr 写入的消息。

### 4.8 repo prune

`repo prune`的基本格式为：

```shell
$ repo prune [project-list]
```

删减（删除）已合并的主题。

### 4.9 repo start

`repo start`的基本格式为：

```shell
$ repo start
<branch-name> [project-list]
```

从清单中指定的修订版本开始，创建一个新的分支进行开发。

`BRANCH_NAME` 参数用于简要说明您尝试对项目进行的更改。如果您不知道，则不妨考虑使用名称 `default`。

`project-list` 参数指定了将参与此主题分支的项目。

***注意：句点 (.) 是一个简写形式，用来代表当前工作目录中的项目。***

### 4.10 repo status

`repo status`的基本格式为：

```shell
$ repo status [project-list]
```

对于每个指定的项目，将工作树与临时区域（索引）以及此分支 (HEAD) 上的最近一次提交进行比较。在这三种状态存在差异之处显示每个文件的摘要行。

要仅查看当前分支的状态，请运行 `repo status`。系统会按项目列出状态信息。对于项目中的每个文件，系统使用两个字母的代码来表示：

在第一列中，大写字母表示临时区域与上次提交状态之间的不同之处。

| 字母 | 含义       | 说明                                       |
| ---- | ---------- | ------------------------------------------ |
| -    | 没有变化   | 在 HEAD 与索引中相同                       |
| A    | 已添加     | 不存在于 HEAD 中，但存在于索引中           |
| M    | 已修改     | 存在于 HEAD 中，但索引中的文件已修改       |
| D    | 已删除     | 存在于 HEAD 中，但不存在于索引中           |
| R    | 已重命名   | 不存在于 HEAD 中，索引中文件的路径已更改   |
| C    | 已复制     | 不存在于 HEAD 中，复制自索引中的另一个文件 |
| T    | 模式已更改 | HEAD 与索引中的内容相同，但模式已更改      |
| U    | 未合并     | HEAD 与索引之间存在冲突；需要加以解决      |

在第二列中，小写字母表示工作目录与索引之间的不同之处。

| 字母 | 含义    | 说明                                       |
| ---- | ------- | ------------------------------------------ |
| -    | 新/未知 | 不存在于索引中，但存在于工作树中           |
| m    | 已修改  | 存在于索引中，也存在于工作树中（但已修改） |
| d    | 已删除  | 存在于索引中，但不存在于工作树中           |

# 5、Manifest

Repo管理的核心就在于Manifest，每个采用repo管理的复杂多仓库项目都需要一个对应的manifest仓库，用来存储所有仓库的配置信息。Repo也是读取此仓库的配置文件来进行管理操作。manifest的配置是xml定义的结构，主要有两个配置：

* 子仓库的仓库地址
* 子仓库详细配置信息

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

其中：

* xml声明，只能放置在xml文件的最前面：

* > ```xml
    > <?xml version="1.0" encoding="UTF-8"?>
    > ```

* manifest：最顶层xml元素：

* > ```xml
    > <manifest>
    >     ...
    > </manifest>
    > ```

* remote元素：设置远程git服务器的属性：

* > ```xml
    > <remote  name="github"
    >          fetch=".."
    >          review="review.cyanogenmod.org" />
    > 
    > <remote  name="private"
    >          fetch="ssh://git@github.com" />
    > 
    > <remote  name="aosp"
    >          fetch="https://android.googlesource.com"
    >          review="android-review.googlesource.com"
    >          revision="refs/tags/android-7.1.1_r6" />
    > ```

    + name：指定远程服务器的名称，直接用于git fetch，git remote等操作，例如，设置远程git服务器为`github`：

    + > ```xml
        > name="github"
        > ```

    + alias：远程git服务器的别名，若有指定，则***覆盖上边name的设定***。***在一个manifest中，name值不能重名，但是alias可以重名***；

    + fetch：git URL，是相关project的git URL前缀；如上边的：

        - 指定仓库下载的位置为本manifest工程的上一级目录：

        - > ```xml
            > fetch=".."
            > ```

        - 指定远程服务器地址为`ssh://git@github.com`：

        - > ```xml
            > fetch="ssh://git@github.com"
            > ```

        - 指定远程服务器地址为`https://android.googlesource.com`：

        - > ```xml
            > fetch="https://android.googlesource.com"
            > ```

            

    + review：指定Gerrit服务器，用于`repo upload`操作，若未指定，则`repo upload`操作无效。如设定gerrit代码审核服务器的位置为`review.cyanogenmod.org`：

    + > ```xml
        > review="review.cyanogenmod.org" />
        > ```

    + revision：指定默认git分支。如设定默认git分支为`refs/tags/android-7.1.1_r6`：

    + > ```xml
        > revision="refs/tags/android-7.1.1_r6"
        > ```

* default元素：指定默认远程git服务器的属性：

* > ```xml
    > <default revision="refs/heads/cm-14.1"
    >          remote="github"
    >          sync-c="true"
    >          sync-j="4" />
    > ```

    + revision：指定默认git分支，如

        - `master`：

        - > ```xml
            > revision="master"
            > ```

        - `refs/heads/cm-14.1`：

        - > ```xml
            > revision="refs/heads/cm-14.1"
            > ```

    + remote：指定默认远程服务器为上边remote元素中声明的服务器，如`github`：

    + > ```xml
        > remote="github"
        > ```

    + sync-c：若设置为true，则只同步指定的分支，即上边的default元素中的revision指定的分支，而不是所有的ref内容：

    + sync-s：若设置为true，则会同步git的子项目；

    + sync-j：指定`repo sync`的默认并行数目，如设置为4：

    + > ```xml
        > sync-j="4"
        > ```

* 注释：

* > ```xml
    > <!-- AOSP Projects -->
    > ```

* project元素：具体子项目信息：

* > ```xml
    > <project path="build" name="CyanogenMod/android_build" groups="pdk,tradefed">
    >     <copyfile src="core/root.mk" dest="Makefile" />
    > </project>
    > <project path="build/blueprint" name="platform/build/blueprint" groups="pdk,tradefed" remote="aosp" />
    > <project path="build/kati" name="CyanogenMod/android_build_kati" groups="pdk,tradefed" />
    > <project path="build/soong" name="platform/build/soong" groups="pdk,tradefed" remote="aosp" >
    >     <linkfile src="root.bp" dest="Android.bp" />
    >     <linkfile src="bootstrap.bash" dest="bootstrap.bash" />
    > </project>
    > 
    > ...
    > 
    > <project path="external/chromium-webview" name="platform/external/chromium-webview" groups="pdk" clone-depth="1" remote="aosp" />
    > ```

    + 工程子项目节点声明：

    + > ```xml
        > <project ... />
        > ```

        若中间包含其它节点，如copyflie节点，则为：

        > ```xml
        > <project ...
        >     <copyfile ... />
        > </project>
        > ```

    + path：可选的路径。指定git clone出来的代码存放在本地的子目录。如果没有指定，则以name作为子目录名。如下载工程代码到本repo工程的build/buleprint目录：

        > ```xml
        > path="build/blueprint"
        > ```

    + name：唯一的名字标识project，同时也用于生成git仓库的URL。格式为`$(remote_fetch)/$(project_name).git`，如上边第一个project的name值为`CyanogenMod/android_build`，未指定remote信息，则该工程代码的git仓库为默认服务器`github`的`../CyanogenMod/android_build.git`；

    + groups：列出project所属的组，以空格或者逗号分隔多个组名。

        - 所有的project都自动属于"all"组；
        - 每一个project自动属于name:'name' 和path:'path'组。例如上边的第三个project指定groups为pdk组和tradefed组：

        - > ```xml
            > <project path="build/kati" name="CyanogenMod/android_build_kati" groups="pdk,tradefed" />
            > ```

            若去掉groups字段，它自动属于default, name:CyanogenMod/android_build_kati和 path:build/kati组；

        - 如果一个project属于notdefault组，则repo sync时不会下载

    + clone-depth：指定git clone的深度，为1表示只clone最近一次commit；

    + remote：指定远程服务器，如第二个project指定远程服务器为`aosp`，即其地址为`https://android.googlesource.com`：

    + > ```xml
        > remote="aosp"
        > ```

    + copyfile/linkfile节点：

        - src：源文件；
        - dest：目标文件；

    + revision： 指定需要获取的git分支，可以是master, refs/heads/master, tag或者SHA-1值；
    + sync_c：如果设置为true，则只同步指定的分支(revision 属性指定)，而不是所有的ref内容；
    + sync_s：如果设置为true，则会同步git的子项目；
    + upstream：在哪个git分支可以找到一个SHA1。用于同步revision锁定的manifest(-c 模式)。该模式可以避免同步整个ref空间；
    + annotation：可以有多个annotation，格式为name-value pair。在repo forall 命令中这些值会导入到环境变量中；
    + remove-project：从内部的manifest表中删除指定的project。经常用于本地的manifest文件，用户可以替换一个project的定义；

* include: 通过name属性可以引入另外一个manifest文件(路径相对与manifest repository's root)：

* > ```xml
    > <include name="snippets/cm.xml" />
    > ```

## 6、用Repo管理工程文件

### 6.1 准备工程仓库

在使用Repo管理工程文件之前，我们需要将相关仓库准备好，可通过git创建。例如，我们git服务器相关资料为：

* git服务器名：`gitserver`

* git服务器地址：`192.168.1.200`

* git账户：`git`

* git仓库根目录：`/home/git/repository`

* 仓库一：目录`repo1`包含仓库：

    - test0.git

    - test1.git

    - test2.git

* 仓库二：目录`repo2`包含仓库：

    - test3.git

    - test4.git

    - repo3：
        + test5.git
        + test6.git
        + test7.git

* manifests.git

其中，

* 仓库一、仓库二和manifests.git均git仓库根目录`/home/git/repository`下；
* 仓库一、仓库二包含的是相关工程代码仓库；
* manifests.git为repo的manifests的仓库，存放工程仓库清单文件`default.xml`；

我们远程登陆到git服务器上：

```shell
$ ssh 192.168.1.200 -l git
git@192.168.1.200's password:
```

输入账户密码后，我们将在Terminal中看到以下字样：

```shell
git@gitserver:~$
```

好了，我们开始创建工程空仓库：

```shell
$ mkdir -p ~/repository/repo1
$ cd ~/repository/repo1
$ git init --bare test0.git
$ git init --bare test1.git
$ git init --bare test2.git

$ mkdir -p ~/repository/repo2
$ cd ~/repository/repo2
$ git init --bare test3.git
$ git init --bare test4.git
$ mkdir -p ~/repository/repo2/repo3/
$ cd ~/repository/repo2/repo3/
$ git init --bare test5.git
$ git init --bare test6.git
$ git init --bare test7.git

$ cd ~/repository
$ git init --bare manifests.git
```



