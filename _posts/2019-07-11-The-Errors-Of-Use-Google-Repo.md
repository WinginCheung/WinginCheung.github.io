---
layout:     post
title:      The Errors of Use Google-Repo
subtitle:   Google-Repo使用错误集锦
date:       2019-07-11
author:     Wingin Cheung
header-img: img/post-google.jpg
catalog: true
mermaid: true
tags:
    - repo

---

# Google-Repo使用错误集锦

# 1、gpg: ***: Permission denied

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

执行Repo时权限出现问题，请删除用户目录下的.repoconfig文件夹、本目录下的.repo文件夹后再次执行`repo init`命令即可：

```shell
$ sudo rm -rf ~/.repoconfig
$ rm -rf .repo
$ repo init -u <manifest-url>
```

## 2、error.GitError: manifests var: *** Please tell me who you are.

```shell
$ repo init -u <manifest-url>

...

error.GitError: manifests var:
*** Please tell me who you are.

Run

    git config --global user.email "you@example.com"
    git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got '<user-name>@<pc-name>.(none)')
```

用户身份未设置，请使用以下指令设置用户身份：

```shell
$ git config --global user.email "you@example.com"
$ git config --global user.name "Your Name"
```

## 3、fatal: error [Errno 101] Network is unreachable

> fatal: Cannot get https://gerrit.googlesource.com/git-repo/clone.bundle
>
> fatal: error [Errno 101] Network is unreachable

指定无服务器无法连接，若是在`repo init`时出现，请参考[`用Google Repo管理多仓库项目之Repo介绍 3.1 修改REPO_URL`](http://wingincheung.github.io/2019/06/13/Manage-Project-With-Google-Repo-About-Repo/)修改REPO_URL，或者直接指定REPO_URL，例如，指定REPO_URL为tuna镜像源：

```shell
$ repo init -u git@192.168.1.200:/home/git/repository/manifests.git --repo-url=https://mirrors.tuna.tsinghua.edu.cn/git/git-repo
```

## 4、fatal: index-pack failed
fatal: early EOF
fatal: index-pack failed

```shell
$ repo sync

...

Fetching project platform/external/cldr
remote: Counting Objects: 8606, done          MiB | 337.00 KiB/s
error: RPC failed; curl 56 GnuTLS recv error (-110): The TLS connection was non-properly terminated.
fatal: The remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed
```

网络不稳定等原因造成的错误，我们可以使用以下脚本进行重试：

```shell
#!/bin/sh

repo sync -f

while [ $? == 1 ]; do
    repo sync -f
done
```

待出现以下字样后，代表同步源码树完成：

```shell
Fetching projects: 100% (730/730)
Fetching projects: 100% (730/730), done.
```

同步完成后，我们可以删除同步失败时产生的临时文件tmp_pack_xxxxxx：

````shell
$ find <repo-dir> -name "tmp_pack_*" | xargs -I {} rm -rf {}
````

## 5、Server does not provide clone.bundle

```shell
$ repo sync

...

curl: (22) The requested URL returned error: 404 Not Found
Server does not provide clone.bundle; ignoring.
```

下载clone.bundle失败，根据google的说明：

> Repo attempts to download a prepackaged bundle file to bootstrap each git prior to downloading the most recent data via Git's HTTP protocol.

在通过 Git 的 HTTP 协议下载最新数据之前，Repo 尝试下载预先打包的捆绑文件以引导每个 git。

> If a bundle file isn't available (like in this case), Repo will ignore it and proceed anyway. In other words, don't pay any attention to this.

如果捆绑文件不可用（如本例所示），Repo 将忽略它并继续进行，换句话说，不要注意这一点。

也就是说，这种错误，无需理会。

若是不想有此错误出现，可取消下载clone.bundle：

```shell
$ repo init -u <manifest-url> --no-clone-bundle
$ repo sync --no-clone-bundle
```

