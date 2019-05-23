---
layout:     post
title:      Linux Module
subtitle:   Module基础知识
date:       2019-05-23
author:     Wingin Cheung
header-img: img/module-head.jpg
catalog: true
mermaid: true
tags:
    - linux
    - module

---

# Linux Module基础知识

## 1、概述

​    模块(module)是一种向Linux内核添加设备驱动程序、文件系统及其他组件的有效方法，而无需连编新内核或重启系统，模块消除了宏内核的许多限制，模块有许多优点：

+ 通过使用模块，内核发布者能够预先编译大量驱动程序，但并不会造成内核镜像发生膨胀，在自动检测硬件(例如USB)或用户提示之后，安装例程选择适当的模块并将其添加到内核中；
+ 内核开发者可以将试验性的代码打包到模块中，模块可以卸载、修改代码或重新打包后可以重新加载，这使得可以快速测试新特性，无需每次都重启系统

    模块几乎可以无缝地插入到内核，模块代码导出一些函数，可以由其他核心模块(包括持久编译到内核中的代码)使用。同样，在模块代码需要卸载时，模块和内核剩余部分之间的关联，也会相应终止。

## 2、module相关命令

### 2.1 module装载

​    module可通过insmod动态装载到运行中的内核中：

```shell
insmod <module>.ko
```

    或者使用modprobe指令动态装载：

```shell
modprobe <module>.ko
```

    ***Tips：insmod不能自动解决module依赖关系，而modprobe可自动添加依赖的module***

### 2.2 module卸载

​    module可通过rmmod从运行的内核中卸载：

```shell
rmmod <module>.ko
```

​    或者使用modprobe指令卸载：

```shell
modprobe -r <module>.ko
```

​    ***Tips：rmmod不能自动解决module依赖关系，而modprobe可自动处理module的依赖关系***

### 2.3 查看已加载的module

​    在终端Terminal中输入以下指令：

```shell
lsmod
```

​    或者cat相关文件查看：

```shll
cat /proc/modules
```

### 2.4 查看module详细信息

​     在终端Terminal中输入以下指令：

```shell
modinfo <module>.ko
```

## 3、module编写及使用

​    话不多说，按照惯例，我们先用module向世界say hello先～

​    首先，我们先建个名为module_hello.c的文件，其中内容如下：

```c
#include <linux/kernel.h>

#include <linux/module.h>


int init_module( void )
{
    printk(KERN_INFO "Hello, World!\n");
    return 0;
}

void cleanup_module( void )
{
    printk(KERN_INFO "Good Bye, World\n");
}

MODULE_LICENSE( "GPL" );
MODULE_AUTHOR( "Wingin Cheung" );
MODULE_DESCRIPTION( "say hello in a module." );
```

​    然后我们再建个名为Makefile的文件，其内容为：

```c
obj-m += module_hello.o

all:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
    make -C /lib/modules/$(shell uname -r)/buile M=$(PWD) clean
```

​    接着我们在终端Terminal中输入编译指令：

```shell
$ make
```

​    当编译完成后，我们可以在当前目录下，看到一个名为module_hello.ko的文件。

​    让我们输入装载module指令见证"奇迹”吧：

```shell
$ sudo insmod module_hello.ko
```

​    什么？什么都没看到？经典的"Hello, World!"呢？？？？？！！！！！

​    肯定是你操作姿势不对，快翻+10086个筋斗～

​    嗯？筋斗翻完了？非常好，让我们看看它去哪里了，来，让系统告诉我们吧：

```shell
$ dmesg
```

​    系统"说"了，你看到了么：

```shell
...
[263479.055856] Hello, World!    
```

​    还不信？那就让系统再告诉我们module_hello.ko是否真的在运行吧：

```shell
$ lsmod | grep module_hello
```

​    系统给你回馈了吧？

```shell
module_hello            16384    0
```

​    "偷窥"完了吧，那我们卸载了它先，让它消失、让系统清净会儿：

```shell
$ rmmod module_hello.ko
```

​    同样，请先翻+10086个筋斗，然后再让系统告诉我们module_hello.ko退出去了：

```shell
[263749.031647] Good Bye, World
```

​    好了，装/卸载模块我们都试过了，让我们"窥探"一下折腾了我们一会儿的module_hello.ko具体的信息：

```shell
$ modinfo module_hello.ko
filename:        /home/wingin/module_hello/module_hello.ko
desrcription:    say hello in a module.
author:          Wingin Cheung
license:         GPL
srcversion:      C1CBAE74E7A6E03AC313841
depends:
retpoline:       Y
name:            module_hello
vermagic:        4.15.0-50-generic SMP mod_unload
```

    什么都看完了，module会玩了吧？那跪安吧，哈哈～