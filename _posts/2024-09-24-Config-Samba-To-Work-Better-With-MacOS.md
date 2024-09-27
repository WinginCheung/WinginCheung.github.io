---
layout:     post
title:      利用Samba实现Linux和MacOS / IOS间的共享文件
subtitle:   samba共享搭建及问题处理
date:       2024-09-24
author:     Wingin Cheung
header-img: img/samba/share.jpg
catalog: true
mermaid: true
tags:
    - samba


---

# 利用Samba实现Linux和MacOS / IOS间的共享文件

测试环境：

+   Ubuntu 22.04.4 LTS
+   IOS 17.2



在我的工作过程中，为了实现多机且多系统间的文件传输，我在个人安装了Ubuntu 22.04的Samba文件共享服务。以下以一个名为 ***ubuntu_share*** 的目录为例实现文件夹共享实现和取消操作。

## 1、Samba安装及文件夹共享配置

在Ubuntu 22.04的终端中，正常已安装Smaba组件。若未安装，我们可以在通过nautilus-share界面来安装。右键点击***ubuntu_share***目录，选择***Local Network Share***，在弹出的***Folder Sharing***界面中，勾选***Share this folder***，若未安装Samba相关组件，则会弹出***Sharing service is not installed***提示：

![install_service_tips.png](/img/samba/install_service_tips.png)

点击***Install service***，弹出samba安装提示：

![install_service_confirm_tips.png](/img/samba/install_service_confirm_tips.png)

点击***install***根据提示输入账号密码开始安装Samba。安装完成后，会自动关闭安装界面，回到***Folder Sharing***界面：

![folder_share_confirm_tips.png](/img/samba/folder_share_confirm_tips.png)

点击***Create Share***即可启用文件夹共享。文件夹共享启动后，图标右下角将增加共享标识：

![folder_share_tips.png](/img/samba/folder_share_tips.png)

## 2、Samba共享文件夹移除

右键点击***ubuntu_share***目录，选择***Local Network Share***，在弹出的***Folder Sharing***界面中，取消***Share this folder***的勾选，并点击 ***Modify Share*** 即可取消文件夹共享。

![folder_share_remove_share_tips.png](/img/samba/folder_share_remove_share_tips.png)

## 3、Samba移除

在终端中，请确认是否已经安装Samba：

```shell
$ samba -V
Version 4.15.13-Ubuntu
```

可以先查询Samba依赖关系：

```shell
$ sudo apt depends samba
samba
  PreDepends: dpkg (>= 1.15.6~)
    dpkg:i386
  PreDepends: init-system-helpers (>= 1.54~)
  Depends: adduser
  Depends: libpam-modules
  Depends: libpam-runtime (>= 1.0.1-11)
  Depends: lsb-base (>= 4.1+Debian)
  Depends: procps
    procps:i386
  Depends: python3 (<< 3.11)
  Depends: python3-dnspython
  Depends: python3-samba
  Depends: samba-common (= 2:4.15.13+dfsg-0ubuntu1.6)
  Depends: samba-common-bin (= 2:4.15.13+dfsg-0ubuntu1.6)
  Depends: tdb-tools
  Depends: python3 (>= 3.10~)
  Depends: <python3:any>
    python3:i386
    python3
  Depends: libbsd0 (>= 0.6.0)
  Depends: libc6 (>= 2.34)
  Depends: libgnutls30 (>= 3.7.0)
  Depends: libldb2 (>= 2:2.4.4-0ubuntu0.22.04.2~)
  Depends: libpopt0 (>= 1.14)
  Depends: libpython3.10 (>= 3.10.0)
  Depends: libtalloc2 (>= 2.3.3~)
  Depends: libtasn1-6 (>= 4.14)
  Depends: libtdb1 (>= 1.4.4~)
  Depends: libtevent0 (>= 0.11.0~)
  Depends: libwbclient0 (= 2:4.15.13+dfsg-0ubuntu1.6)
  Depends: samba-libs (= 2:4.15.13+dfsg-0ubuntu1.6)
  Recommends: attr
    attr:i386
  Recommends: logrotate
  Recommends: python3-markdown
  Recommends: samba-dsdb-modules
  Recommends: samba-vfs-modules
  Suggests: bind9 (>= 1:9.5.1)
  Suggests: bind9utils
    bind9-utils
  Suggests: ctdb
  Suggests: ldb-tools
 |Suggests: ntp
  Suggests: chrony (>= 3.0-1)
  Suggests: smbldap-tools
  Suggests: ufw
  Suggests: winbind
  Enhances: bind9
  Enhances: ntp
```

查询已安装的Smaba相关组件：

```shell
$ sudo apt list --installed | grep samba

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

python3-samba/jammy-updates,now 2:4.15.13+dfsg-0ubuntu1.6 amd64 [installed,automatic]
samba-common-bin/jammy-updates,now 2:4.15.13+dfsg-0ubuntu1.6 amd64 [installed,automatic]
samba-common/jammy-updates,jammy-updates,now 2:4.15.13+dfsg-0ubuntu1.6 all [installed]
samba-dsdb-modules/jammy-updates,now 2:4.15.13+dfsg-0ubuntu1.6 amd64 [installed,automatic]
samba-libs/jammy-updates,now 2:4.15.13+dfsg-0ubuntu1.6 amd64 [installed,automatic]
samba-vfs-modules/jammy-updates,now 2:4.15.13+dfsg-0ubuntu1.6 amd64 [installed,automatic]
samba/jammy-updates,now 2:4.15.13+dfsg-0ubuntu1.6 amd64 [installed]
```

移除samba相关组件：

```shell
$ sudo apt purge -y samba samba-common samba-vfs-modules
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  apg attr gnome-control-center-faces gnome-online-accounts ldap-utils libbasicobjects0 libcephfs2 libcollection4
  libcolord-gtk1 libdhash1 libgsound0 libgssdp-1.2-0 libgupnp-1.2-1 libgupnp-av-1.0-3 libgupnp-dlna-2.0-4 libini-config5
  libipa-hbac0 libldb2 libnfsidmap1 libnss-sss libpath-utils1 libref-array1 librygel-core-2.6-2 librygel-db-2.6-2
  librygel-renderer-2.6-2 librygel-server-2.6-2 libsss-certmap0 libsss-idmap0 libsss-nss-idmap0
  mobile-broadband-provider-info network-manager-gnome python3-certifi python3-dnspython python3-gpg python3-ldb
  python3-macaroonbakery python3-protobuf python3-pymacaroons python3-requests python3-rfc3339 python3-samba python3-sss
  python3-talloc python3-tdb rygel samba-dsdb-modules samba-libs sssd-common sssd-krb5 sssd-krb5-common sssd-ldap
  sssd-proxy tdb-tools
Use 'sudo apt autoremove' to remove them.
The following packages will be REMOVED:
  samba* samba-common* samba-common-bin* samba-vfs-modules*
0 upgraded, 0 newly installed, 4 to remove and 68 not upgraded.
After this operation, 21.7 MB disk space will be freed.
(Reading database ... 354001 files and directories currently installed.)
Removing samba (2:4.15.13+dfsg-0ubuntu1.6) ...
Removing samba-common-bin (2:4.15.13+dfsg-0ubuntu1.6) ...
Removing samba-common (2:4.15.13+dfsg-0ubuntu1.6) ...
Removing samba-vfs-modules:amd64 (2:4.15.13+dfsg-0ubuntu1.6) ...
Processing triggers for man-db (2.10.2-1) ...
Processing triggers for libc-bin (2.35-0ubuntu3.8) ...
(Reading database ... 353665 files and directories currently installed.)
Purging configuration files for samba-common (2:4.15.13+dfsg-0ubuntu1.6) ...
Purging configuration files for samba (2:4.15.13+dfsg-0ubuntu1.6) ...
Processing triggers for ufw (0.36.1-4ubuntu0.1) ...
```

删除Samba相关配置及文件夹：

```shell
$ sudo rm -rf /var/cache/samba /etc/samba /run/samba /var/lib/samba /var/log/samba
```

删除sambashare组：

```shell
$ sudo delgroup sambashare
```

## 4、可能出现的问题

### 4.1 远程无访问权限

使用指定用户连接到samba服务器时，出现提示 ***You entered an invalid username or password for the server. Authentication error***.

![authentication_error.png](/img/samba/authentication_error.png)

查看指定用户是否在sambashare组：

```shell
$ cat /etc/group | grep samba
sambashare:x:132:
```

添加指定用户到sambashare组：

```shell
$ sudo usermod -aG sambashare <user-name>
```

修改指定用户的samba访问密码（根据提示输入密码）：

```shell
$ sudo smbpasswd -a <user-name>
New SMB password:
Retype new SMB password:
Added user <user-name>.
```

### 4.2 MacOS / IOS无法向共享目录写入文件

MacOS / IOS中拷贝文件到共享目录时，会出现 ***The Operation Can't Be Completed.*** 的提示，具体提示中会字段中出现 ***because its name is too long or includes characters that are invalid on the destination volume.*** 字样。

![copy_error.png](/img/samba/copy_error.png)

根据Samba Wiki中[Configure Samba to Work Better with Mac OS X](https://wiki.samba.org/index.php/Configure_Samba_to_Work_Better_with_Mac_OS_X#:~:text=Below%20are%20suggested%20parameters%20to%20use%20in%20smb.conf%20file%20of)一文，我们需要对samba配置作小小的改动。

安装fruit和samba-vfs-modules：

```shell
$ sudo apt install fruit samba-vfs-modules
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  apg gnome-control-center-faces gnome-online-accounts ldap-utils libbasicobjects0 libcollection4 libcolord-gtk1 libdhash1
  libgsound0 libgssdp-1.2-0 libgupnp-1.2-1 libgupnp-av-1.0-3 libgupnp-dlna-2.0-4 libini-config5 libipa-hbac0 libnfsidmap1
  libnss-sss libpath-utils1 libref-array1 librygel-core-2.6-2 librygel-db-2.6-2 librygel-renderer-2.6-2
  librygel-server-2.6-2 libsss-certmap0 libsss-idmap0 libsss-nss-idmap0 mobile-broadband-provider-info
  network-manager-gnome python3-macaroonbakery python3-protobuf python3-pymacaroons python3-rfc3339 python3-sss rygel
  sssd-common sssd-krb5 sssd-krb5-common sssd-ldap sssd-proxy
Use 'sudo apt autoremove' to remove them.
Suggested packages:
  scid xboard
The following NEW packages will be installed:
  fruit samba-vfs-modules
0 upgraded, 2 newly installed, 0 to remove and 68 not upgraded.
Need to get 736 kB of archives.
After this operation, 2,596 kB of additional disk space will be used.
Get:1 http://mirrors.ustc.edu.cn/ubuntu jammy/universe amd64 fruit amd64 2.1.dfsg-9 [317 kB]
Get:2 http://mirrors.ustc.edu.cn/ubuntu jammy-updates/main amd64 samba-vfs-modules amd64 2:4.15.13+dfsg-0ubuntu1.6 [419 kB]
Fetched 736 kB in 1s (1,027 kB/s)      
Selecting previously unselected package fruit.
(Reading database ... 353893 files and directories currently installed.)
Preparing to unpack .../fruit_2.1.dfsg-9_amd64.deb ...
Unpacking fruit (2.1.dfsg-9) ...
Selecting previously unselected package samba-vfs-modules:amd64.
Preparing to unpack .../samba-vfs-modules_2%3a4.15.13+dfsg-0ubuntu1.6_amd64.deb ...
Unpacking samba-vfs-modules:amd64 (2:4.15.13+dfsg-0ubuntu1.6) ...
Setting up fruit (2.1.dfsg-9) ...
Setting up samba-vfs-modules:amd64 (2:4.15.13+dfsg-0ubuntu1.6) ...
Processing triggers for man-db (2.10.2-1) ...
```

在samba配置文件 ***/etc/samba/smb.conf*** 的 ***[global]*** 字段中添加以下内容：

```text
#======================= Global Settings =======================

[global]
min protocol = SMB2
ea support = yes
vfs objects = fruit streams_xattr
```

移除共享目录并重新设置目录共享即可（主要是为了重新加载samba配置文件）。
