---
layout:     post
title:      Petalinux 2018.2 for Xilinx
subtitle:   Petalinux 2018.2 for Xilinx的安装、使用与优化
date:       2019-04-04
author:     Wingin Cheung
header-img: img/cloud-bg.jpg
catalog: true
tags:
    - Petalinux
    - Xilinx
---

# Petalinux 2018.2 for Xilinx

## 1、概述

​	Petalinux是Xilinx公司推出的嵌入式Linux开发套件，包括了Linux Kernel、u-boot、device-tree、rootfs等源码、库，以及Yocto recipes，可以让客户很方便的生成、配置、编译及自定义。Petalinux支持Zynq UltraScale+ MPSoC、Zynq-7000全可编程SoC，以及MicroBlaze，可与Xilinx硬件设计工具Vivado协同工作，大大简化了Linux系统的开发工作。

​	使用PetaLinux工具，开发人员可以定制u-boot、Linux内核或Linux应用，开发者还可以通过网络或JTAG在随附的全系统仿真器 (QEMU) 或物理硬件上添加新的内核、器件驱动程序、应用和库，以及启动并测试软件协议栈，完成从系统启动到执行的所有操作。在主机端提供的PetaLinux工具包括：

 - 命令行界面
 - 应用、器件驱动程序、库生成器以及开发模板
 - 可引导的系统镜像生成器
 - 调试代理程序
 - GCC工具集
 - 集成的QEMU全系统仿真器
 - 自动化工具
 - 支持Xilinx系统调试器

## 2、安装Petalinux

### 2.1 安装需求

​	Petalinux是个大型软件，对电脑硬件配置要求比较高。Petalinux工具用户文档[UG1144(v2018.2)](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2018_2/ug1144-petalinux-tools-reference-guide.pdf) Page 9对安装环境做了个推荐：

> - 8 GB RAM (recommended minimum for Xilinx tools)
> - 2 GHz CPU clock or equivalent (minimum of 8 cores
> - 100 GB free HDD space
> - Supported OS: 
>>-  Red Hat Enterprise Workstation/Server 7.2, 7.3, 7.4 (64-bit) 
>>- CentOS 7.2, 7.3, 7.4 (64-bit) 
>>- Ubuntu Linux 16.04.3 (64-bit) 

​	为安装Petalinux，我们需要8GB的内存、2GB主频的CPU、100GB的硬盘，还需要一个能正常运行的Linux系统。

​	接下来，我们将在[Ubuntu 16.04.6 LTS (Xenial Xerus)](http://releases.ubuntu.com/16.04/)桌面版本上进行Petalinux的安装使用。其它版本系统，部分指令或者配置可能不兼容，请参考相关系统文档。

### 2.2 安装依赖库

​	Petalinux的运行依赖于一些库，根据Petalinux工具用户文档[UG1144(v2018.2)](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2018_2/ug1144-petalinux-tools-reference-guide.pdf) Page 9中Table 2-1：Packages and Linux Workstation Environments一表，整理了一下需要安装的库，做成了一个自动安装脚本install_petalinux201802_lib.sh：

```shell
#!/bin/bash
sudo apt -y install tofrodos  iproute2 gawk
sudo apt -y install gcc git make
sudo apt -y install xvfb
sudo apt -y install net-tools  libncurses5-dev  tftpd
sudo apt -y install zlib1g-dev zlib1g-dev:i386 libssl-dev  flex bison libselinux1
sudo apt -y install gnupg wget diffstat chrpath socat xterm
sudo apt -y install autoconf libtool tar unzip texinfo zlib1g-dev gcc-multilib build-essential libsdl1.2-dev libglib2.0-dev
sudo apt -y install screen pax gzip tar
```

​	建议安装依赖库之前先更新系统及其库等，确保所有软件在最新状态。

### 2.3 安装TFTP软件

​	TFTP软件用于通过网络在线更新系统、软件等，我们在主板调试时将会用到。以下为TFTP的自动安装脚本instll_tftp.sh：

```shell
#!/bin/bash

configfile="/etc/default/tftpd-hpa"
sudo apt install -y tftpd-hpa tftp-hpa
sudo cp ${configfile} /etc/default/tftpd-hpa.bck
sudo sed -i '/^TFTP_OPTION/d' ${configfile}
sudo sed -i '$a TFTP_OPTION=\"--secure --create\"' ${configfile}
sudo chown -R tftp /var/lib/tftpboot/
echo "export TFTP_DIRECTORY=\`cat /etc/default/tftpd-hpa | grep TFTP_DIRECTORY | cut -d \"\\\"\" -f 2\`" | sudo tee -a /etc/bash.bashrc >/dev/null
sudo systemctl enable tftpd-hpa
sudo systemctl restart tftpd-hpa
```

### 2.4 安装文件下载

​	Petalinux可在[Xilinx官网](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools/2018-2.html)免费[下载](https://www.xilinx.com/member/forms/download/xef.html?filename=petalinux-v2018.2-final-installer.run)，但需先注册账号方可下载，安装包大小为6.15G。

​	为了确保下载的安装包下载完整，请对其md5进行校验：

```shell
#/bin/bash
echo "686edec30123bacf94102f2bc6ed70ff  petalinux-v2018.2-final-installer.run" > petalinux-v2018.2-final-installer.md5
md5sum -c petalinux-v2018.2-final-installer.md5
```

​	以上校验，系统终端中，将输出校验ok信息：

```shell
petalinux-v2018.2-final-installer.run: OK
```

### 2.5 安装

​	根据Petalinux工具用户文档[UG1144(v2018.2)](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2018_2/ug1144-petalinux-tools-reference-guide.pdf) Page 12中的要求：

>Note: You cannot install the tool with the root user, instead the permissions for
>/opt/pkg/petalinux should be 755. It is not mandatory to install tool in /opt/pkg/petalinux
>directory. You can install at any desired location that has the 755 permissions.

​	我们需要使用非root用户安装Petalinux。建议安装Petalinux到个人用户目录下，例如，安装到~/bin/petalinux201802目录下，我们使用终端进入petalinux-v2018.2-final-installer.run所在文件夹，执行以下命令：

```shell
mkdir -p ~/bin/petalinux201802
chmod +x petalinux-v2018.2-final-installer.run
./petalinux-v2018.2-final-installer.run ~/bin/petalinux201802
```

​	安装期间，将会有PetaLinux End User License Agreement (EULA)提示，需要按键盘q，然后按y进行协议许可确认。

### 2.6 环境配置

#### 2.6.1 配置环境变量

​	Petalinux安装完成后，需对其启动环境进行配置：

```shell
source ~/bin/petalinux201802/settings.sh
```

​	但此指令只在当前终端生效，重开终端后，仍需要再次执行此指令。为避免此情况，在Ubuntu系统下，我们也可以将其写入用户配置信息中：

```shell
echo "source ~/bin/petalinux201802/settings.sh" >> ~/.bashrc
```

​	注：1、*在CentOS中，不可将其写入用户配置信息，否则可能会引起登陆用户时系统异常；*

​		2、*此指令未在Red Hat中测试；*

​	Petalinux环境变量生效后，我们可以使用一下指令验证Petalinux的安装情况：

```shell
echo $PETALINUX
```

​	系统将输出Petalinux的安装目录，表示安装成功：

```shell
/home/<user>/bin/petalinux201802
```

​	其中，<user>为安装petalinux的当前用户名。

#### 2.6.2 关闭webtalk功能

​	默认情况下，启用webtalk选项可将工具使用情况统计信息发送回Xilinx，我们可以通过运行petalinux-util --webtalk命令来关闭webtalk功能：

```shell
petalinux-util --webtalk off
```

​	系统将在终端输出一下信息，代表webtalk功能关闭成功：

```shell
INFO: Turn off webtalk feature!
```

#### 2.6.3 修改默认shell为bash

​	Ubuntu默认shell为dash，而在Petalinux工具用户文档[UG1144(v2018.2)](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2018_2/ug1144-petalinux-tools-reference-guide.pdf) Page 14中，要求：

>This section assumes that the following prerequisites have been satisfied: 
>- ...
>- *"/bin/sh" is bash*

​	所以我们需要将我们需要将/bin/sh调整为bash，执行以下指令即可：

```shell
sudo rm /bin/sh
sudo ln -s /bin/bash /bin/sh
```

​	确认是否修改成功，我们可以执行以下指令：

```shell
ls -l /bin/sh
```

​	系统将输出包含以下字段的消息，确认shell已修改成功：

```shell
/bin/sh -> /bin/bash
```

## 3、Petalinux使用

### 3.1 创建工程

​	根据Petalinux工具用户文档[UG1144(v2018.2)](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2018_2/ug1144-petalinux-tools-reference-guide.pdf) Page 17中*Chapter 3 Creating a Project*，Petalinux创建工程有两种方式，分别为：

+ 基于现有BSP文件的工程创建

+ 基于Vivado产生的硬件描述文件的工程创建

  下边，我们分别说明这两种方式的具体指令：

#### 3.1.1 基于现有BSP文件的工程创建

​	基于现有BSP文件的工程创建，需要先从[Xilinx官网](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools/2018-2.html)下载SoC相应的BSP包，或者从别处得到相应的BSP包。

​	当我们得到BSP包后，我们可以开始创建我们的工程。例如，我们在/home/user/project目录下创建工程，可以使用以下指令：

```shell
cd ~/project
petalinux-create -t project -n <PROJECT-NAME> -s <PATH-TO-BSP>
```

​	其中，

+ <PROJECT-NAME>为工程名称，可省略；
+ <PATH-TO-BSP>为BSP包所在目录路径；

#### 3.1.2 基于Vivado产生的硬件描述文件的工程创建

​	基于Vivado产生的硬件描述文件的工程创建，需要先从Vivado导出相应硬件描述文件，具体操作为：

​	1、使用Vivado打开相应工程；

​	2、在选项栏中选择File -> Export -> Export Hardware，勾选☑️Include bitstream选项，然后点击OK即可；

​	Vivado硬件描述文件将生成在Vivado工程的<VIVADO-PROJECT-NAME>.sdk目录下，我们可以使用以下指令创建工程：

```shell
petalinux-create --t project --template <CPU-TYPE> -n <PROJECT-NAME>
```

​	其中，

+ <CPU-TYPE>为CPU类型，具体值可为：*zynqMP (for UltraScale+ MPSoC)*、*zynq (for Zynq)*、*microblaze (for MicroBlaze)*；

+ <PROJECT-NAME>为工程名称；

  工程创建成功后，我们需要根据Vivado硬件描述文件使用petalinux-config配置工程：

```shell
petalinux-config --get-hw-description=<PATH-TO-HDF/DSA-DIRECTORY>
```

​	其中，<PATH-TO-HDF/DSA-DIRECTORY>为Vivado硬件描述文件所在目录路径；

​	根据Vivado硬件描述文件使用petalinux-config配置工程后，我们还需要配置SoC型号CONFIG_SUBSYSTEM_MACHINE_NAME：petalinux-config -> DTG Settings ---> (template) MACHINE_NAME，该值可为：

+ ac701-full
+ ac701-lite
+ kc705-full
+ kc705-lite
+  kcu105
+ zc1254-reva
+ zc1275-reva
+ zc1275-revb
+ zc1751-dc1
+  zc1751-dc2
+ zc702
+ zc706
+ avnet-ultra96-rev1
+ zcu100-reva
+ zcu100-revb
+  zcu100-revc
+ zcu102-rev1.0
+ zcu102-reva
+ zcu102-revb
+ zcu104-reva
+ zcu104-revc
+ zcu106-reva
+ zcu111-reva
+ zedboard

### 3.2 配置工程

​	Petalinux工程创建完成后，我们需要根据具体情况对工程进行相关配置，具体指令为：

+ 全局配置：petalinux-config
+ 配置Kernel：petalinux-config -c kernel
+ 配置rootfs：petalinux-config -c rootfs

### 3.3 编译工程

​	配置完成后，我们可以对工程执行编译工作，具体指令为：

```shell
petalinux-build
```

​	*petalinux-build*指令可包含参数，具体可参考文档[UG1157 PetaLinux Command Line Reference](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2018_2/ug1157-petalinux-tools-command-line-guide.pdf) Page 12 *Table 1-6: petalinux-build Command Line Options*相关内容。

### 3.4 打包文件

​	编译完成后，我们可以打包一些文件，例如启动文件BOOT.bin、预建文件等用于测试的文件。

#### 3.4.1 打包启动文件BOOT.bin

​	启动文件BOOT.bin，可烧录到flash中，将在SoC从flash启动时运行。根据不同的SoC，我们选择不同的指令。

​	for Zynq-7000：

```shell
petalinux-package --boot --fsbl <FSBL-image> --fpga <FPGA-bitstream> --u-boot
```

​	for MicroBlaze：

```shell
petalinux-package --boot --fpga <FPGA-bitstream> --u-boot --kernel
```

​	for Zynq UltraScale+ MPSoC：

```shell
petalinux-package --boot --format BIN --fsbl <FSBL-image> --u-boot <u-boot-image> --pmufw <pmufw-image> --fpga <FPGA-bitstream>
```

#### 3.4.2 打包预建文件

​	预建文件，用于Jtag或者QEMU测试时使用。打包预建文件，我们可以使用以下指令：

```shell
petalinux-package --prebuilt --fpga <FPGA-bitstream>
```

### 3.5 调试

​	预建文件打包好后，我们可以通过petalinux-boot指令使用Jtag在线调试，或者从QEMU启动进行调试。

#### 3.5.1 启动等级

​	对于petalinux-boot指令，有可选选项--prebuild <BOOT_LEVEL>参数，<BOOT_LEVEL>值可为1、2、3三种，其意义分别为：

> + Level 1: Download the prebuilt FPGA bitstream. 
>>  It will also boot FSBL for Zynq-7000 and, FSBL and PMU firmware for Zynq UltraScale+ MPSoC. 
>>  
> + Level 2: Download the prebuilt FPGA bitstream and boot the prebuilt U-Boot. 
>>   For Zynq-7000: It will also boot FSBL before booting U-Boot. 
>>   
>>   For Zynq UltraScale+ MPSoC: It will also boot PMU firmware, FSBL, and ATF before booting U-Boot. 
> + Level 3: 
>>   For MicroBlaze: Downloads the prebuilt FPGA bitstream and boot the prebuilt kernel image on target. 
>>   
>>   For Zynq-7000: Downloads the prebuilt FPGA bitstream and FSBL and boot the prebuilt U-Boot and boot the prebuilt kernel on target. 
>>   
>>   For Zynq UltraScale+ MPSoC: Downloads PMU Firmware, prebuilt FSBL, prebuilt kernel, prebuilt FPGA bitstream, linux-boot.elf and the prebuilt ATF on target. 

#### 3.5.2 使用Jtag在线调试

​	连接好主板、调试器与电脑后，我们可以使用以下指令进行在线调试：

```shell
petalinux-boot --jtag --prebuild 3
```

#### 3.5.3 使用QEMU调试

​	QEMU是一套集成在Petalinux工具集中的模拟处理器，无需硬件环境即可模拟测试，但测试结果仅供参考，与实际环境可能不符。

​	我们可以使用以下指令进行QEMU调试：

```shell
petalinux-boot --qemu --prebuilt 3
```

### 3.6 定制系统

​	在Petalinux工程中，我们可以添加我们所需要的文件，例如库、应用程序、自启动程序、模块或者自定义设备树等，将其加入到我们的Petalinux系统当中。

#### 3.6.1 添加库

​	对于添加库，我们需要先创建一个库应用：

```shell
petalinux-create -t apps --template install --name <LIB-NAME> --enable
```

​	该库所在目录路径为：<plnx-proj-root>/project-spec/meta-user/recipes-apps/<LIB-NAME>。

​	我们可以将我们库的源文件或者预编译好的库文件拷贝到该库所在目录路径下的files目录下，然后编辑该库所在目录路径下的<LIB-NAME>.bb文件，将files目录下需要的文件加入到SRC_URI字段中。如加入源文件lib.c、lib.h、Makefile文件：

```makefile
#
# This file is the libs recipe.
#

SUMMARY = "Simple libs application"
SECTION = "PETALINUX/apps"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

S = "${WORKDIR}"

TARGET_CC_ARCH += "${LDFLAGS}"

SRC_URI = "file://lib.c \
	file://lib.h \
	file://Makefile \
	"

do_compile() {
  	oe_runmake
}

do_install() {
	install -d ${D}${libdir}
	install -m 0655 ${S}/<LIB-NAME>.so ${D}${libdir}
}

FILES_${PN} += "${libdir}"
FILES_SOLIBSDEV = ""
```

​	或者加入预编译库文件<LIB-NAME>.so：

```shell
#
# This file is the libs recipe.
#

SUMMARY = "Simple libs application"
SECTION = "PETALINUX/apps"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

S = "${WORKDIR}"

TARGET_CC_ARCH += "${LDFLAGS}"

SRC_URI = "file://<LIB-NAME>.so \
	"

do_install() {
	install -d ${D}${libdir}
	install -m 0655 ${S}/<LIB-NAME>.so ${D}${libdir}
}

FILES_${PN} += "${libdir}"
FILES_SOLIBSDEV = ""
```

#### 3.6.2 添加应用程序

​	对于添加应用程序，我们需要先创建一个应用程序：

```shell
petalinux-create -t apps --template install --name <APPS-NAME> --enable
```

​	该应用程序所在目录路径为：<plnx-proj-root>/project-spec/meta-user/recipes-apps/<APPS-NAME>。

​	我们可以将我们应用程序的源文件或者预编译好的应用程序文件拷贝到该库所在目录路径下的files目录下，然后编辑该应用程序所在目录路径下的<APPS-NAME>.bb文件，将files目录下需要的文件加入到SRC_URI字段中。如加入源文件apps.c、apps.h、Makefile文件：

```makefile
#
# This file is the apps recipe.
#

SUMMARY = "Simple apps application"
SECTION = "PETALINUX/apps"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

S = "${WORKDIR}"

SRC_URI = "file://apps.c \
	file://apps.h \
	file://Makefile \
	"

do_compile() {
	oe_runmake
}

do_install() {
	install -d ${D}${bindir}
	install -m 0755 ${S}/<APPS-NAME> ${D}${bindir}
}
```

​	或者加入预编译应用程序文件<APPS-NAME>：

```makefile
#
# This file is the arp recipe.
#

SUMMARY = "Simple apps application"
SECTION = "PETALINUX/apps"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

SRC_URI = "file://<APPS-NAME> \
					"

S = "${WORKDIR}"
INSANE_SKIP_${PN} = "ldflags"
INHIBIT_PACKAGE_DEBUG_SPLIT = "1"
INHIBIT_PACKAGE_STRIP = "1"

do_install() {
	install -d ${D}/${bindir}
	install -m 0755 ${S}/<APPS-NAME> ${D}/${bindir}
}
```

#### 3.6.3 添加自启动脚本

​	对于添加自启动脚本，我们需要先创建一个脚本程序：

```shell
petalinux-create -t apps --template install --name <SCRIPT-NAME> --enable
```

​	该脚本所在目录路径为：<plnx-proj-root>/project-spec/meta-user/recipes-apps/<SCRIPT-NAME>。

​	我们在该脚本所在目录路径下的files目录下，增加脚本文件，如autorun-script：

```shell
#!/bin/sh

DAEMON=/usr/bin/autorun-script

start ()
{
	echo " Starting autorun-script"
	start-stop-daemon -S -o --background -x $DAEMON
}

stop () 
{
	echo " Stoping autorun-script"
	start-stop-daemon -K -x $DAEMON
}

restart()
{
	stop
	start
}

[ -e $DAEMON ] || exit 1
	case "$1" in
		start)
			start; ;;
		stop)
			stop; ;;
		restart)
			restart; ;;
		*)
			exit 1
			echo "Usage: $0 {start|stop|restart}"
	esac
exit $?
```

​	编辑该脚本所在目录路径下的<SCRIPT-NAME>.bb文件，将files目录下需要的文件加入到SRC_URI字段中：

```makefile
#
# This file is the script recipe.
#

SUMMARY = "Simple script application"
SECTION = "PETALINUX/apps"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

SRC_URI = "file://autorun-script \
	"

S = "${WORKDIR}"

FILESEXTRAPATHS_prepend := "${THISDIR}/files:"

inherit update-rc.d

INITSCRIPT_NAME = "autorun-script"
INITSCRIPT_PARAMS = "start 99 S ."

do_install() {
	install -d ${D}/${sysconfdir}/init.d
	install -m 0755 ${S}/autorun-script ${D}/${sysconfdir}/init.d/autorun-script
}

FILES_${PN} += "${sysconfdir}/*"
```

#### 3.6.4 添加模块

​	对于添加模块，我们需要先创建一个模块应用：

```shell
petalinux-create -t apps --template install --name <MODULE-NAME> --enable
```

​	该库所在目录路径为：<plnx-proj-root>/project-spec/meta-user/recipes-apps/<MODULE-NAME>。

​	我们可以将我们模块的源文件或者预编译好的库文件拷贝到该模块所在目录路径下的files目录下，然后编辑该模块所在目录路径下的<MODULE-NAME>.bb文件，将files目录下需要的文件加入到SRC_URI字段中。如加入源文件module.c、module.h、Makefile文件：

```makefile
#
# This file is the module recipe.
#

SUMMARY = "Simple shivamod application"
SECTION = "PETALINUX/apps"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

S = "${WORKDIR}"

SRC_URI = "file://module.c \
	file://module.h \
	file://Makefile \
	"

do_compile() {
	oe_runmake
}

do_install() {
	install -d ${D}${base_libdir}/modules/${KERNEL_VERSION}/extra
	install -m 0755 ${S}/<MODULE-NAME>.ko ${D}${base_libdir}/modules/${KERNEL_VERSION}/extra
}

FILES_${PN} = "${base_libdir}/modules/"
```

​	或者加入预编译模块<MODULE-NAME>.ko：

```makefile
#
# This file is the module recipe.
#

SUMMARY = "Simple shivamod application"
SECTION = "PETALINUX/apps"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

SRC_URI = "file://<MODULE-NAME>.ko \
	"

inherit module-base

S = "${WORKDIR}"

do_install() {
	install -d ${D}${base_libdir}/modules/${KERNEL_VERSION}/extra
	install -m 0755 ${S}/<MODULE-NAME>.ko ${D}${base_libdir}/modules/${KERNEL_VERSION}/extra
}

FILES_${PN} = "${base_libdir}/modules/"
```

#### 3.6.5 自定义设备树

​	在Petalinux默认设备树文件的基础上自定义设备树，需要在<plnx-proj-root>/project-spec/meta-user/recipes-bsp/device-tree/目录下进行自定义。该路径下所有的自定义设备树相关字段，将插入或者覆盖原有默认设备树中的相关字段。

​	以下内容为在Zynq添加Microchip的KSZ9897R网卡芯片设备树信息为例，进行示范：

​	1、硬件连接：

+ 通过Zynq的I2C1控制KSZ9897R的I2C接口；
+ 通过Zynq的gem1与KSZ9897R的Port7相连；

​	2、在<plnx-proj-root>/project-spec/meta-user/recipes-bsp/device-tree/files目录下，添加KSZ9897R的相关设备树信息文件ksz9897r-conf-i2c.dtsi，  以下为其具体内容：

```txt
/ {
	amba: amba {
		gem1: ethernet@e000c000 {
			fixed-link {
				speed = <1000>;
				full-duplex;
			};
		};

		i2c1: i2c@e0005000 {
			clock-frequency = <38000>;
			ksz9897r: ksz9897r@0 {
				reg = <0x5f>;
			};
		};
	};
};
```

​	3、修改<plnx-proj-root>/project-spec/meta-user/recipes-bsp/device-tree/files目录下的system-user.dtsi文件，添加ksz9897r-conf-i2c.dtsi相关字段：

```txt
/include/ "system-conf.dtsi"
/include/ "ksz9897r-conf-i2c.dtsi"
/ {
};
```

​	4、修改<plnx-proj-root>/project-spec/meta-user/recipes-bsp/device-tree/目录下的device-tree.bbappend文件，添加ksz9897r-conf-i2c.dtsi相关字段：

```makefile
SRC_URI += "file://system-user.dtsi \
	file://ksz9897r-conf-i2c.dtsi \
	"
```

## 4、Petalinux系统优化

### 4.1 u-boot读取kernel的速度优化

​	系统启动时，u-boot从QSPI flash读取kernel文件的速度是默认的。根据所用的QSPI flash，我们可以调整其读取速度。

​	在<plnx-proj-root>/project-spec/meta-plnx-generated/recipes-bsp/u-boot/configs/platform-auto.h中，在cp_kernel2ram中添加读取速度：

```c
cp_kernel2ram=sf probe 0 <SPEED_HZ> & ...
```

​	其中，<SPEED_HZ>值为Hz计算，宜根据QSPI flash相关参数选择合适的数值，过小读取速度太慢，过大则会造成读取的数据出错；

​	但是以上优化可能会在执行petalinux-config后被修改。如果是我们使用的QSPI flash均为同款flash，我们可以修改Petalinux安装目录下的u-boot_bsp.tcl文件（文件路径文件路径<petalinux-path>/etc/hsm/scripts/libs/）第483行左右：

```c
...

"kernel" {
	append data "\n" { "cp_kernel2ram=sf probe 0 <SPEED_HZ> && sf read ${netstart} ${kernelstart} ${kernelsize}\0" \ }
}

...
```

​	我们将此读取速度配置作为我们以后Petalinux工程的默认配置。当在项目中执行petalinux-config时，此配置将会对项目中原有的platform-auto.h文件进行修改。

### 4.2 关闭ssh-dropbear功能

​	dropbear是一个相对较小的SSH服务器和客户端，在一般情况下，我们无需用到此功能，则可关闭此功能。

​	我们进入rootfs配置环境中，先将package group-core-ssh-dropbear功能关闭：petalinux-config -c rootfs —> Filesystem Packages —> misc —> packagegroup-core-ssh-dropbear，去除packagegproup-core-ssh-dropbear的勾选，然后在<plnx-proj-root>/project-spec/meta-user/conf/petalinuxbsp.conf文件最后增加一行即可：

```make
PACKAGE_EXCLUDE += " packagegroup-core-ssh-dropbear"
```
