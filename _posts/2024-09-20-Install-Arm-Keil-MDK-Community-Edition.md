---
layout:     post
title:      Keil MDK社区版安装
subtitle:   Keil MDK社区版安装
date:       2024-09-20
author:     Wingin Cheung
header-img: img/MDK-Community_Edition/uvision-screenshot.WhichToolHeroImage.png
catalog: true
mermaid: true
tags:
    - mdk

---

# Keil MDK社区版安装

Keil MDK是MCU嵌入式软件开发工程师喜欢使用的IDE之一，但由于资费等原因，很多时候使用了盗版（或者说非法激活？）。好消息是，作为嵌入式软件工程师，现在可以正大光明的使用***免费的完整版Keil MDK***啦，现在我们来参考[Arm Keil官网](https://www.keil.arm.com/mdk-community/)看看如何安装。

***注：Keil MDK社区版仅限电子爱好者、学生等非商业免费评估和使用，不建议用于商业用途！！！！！***

## 1、下载

Arm Keil MDK社区版目前（2024-09-20）最新版本为V5.41.0.0，下载地址为[https://www.keil.com/demo/eval/arm.htm](https://www.keil.com/demo/eval/arm.htm)

## 2、安装

在非管理员权限下双击MDK安装包，按安装界面提示安装即可。

## 3、激活

在非管理员权限下，双击Keil μVision图标，按以下步骤激活MDK：

1）选择***File -> License Management...***，在***License Management***管理界面中选择***User-Based License***选项卡；

![the User-Based License tab](/img/MDK-Community_Edition/uvision-license-management.3244f804bc5b.png)



2）在***User-Based License***选项卡中，点击左下角的***Activate / Deactive...***打开***Arm License Management Utility***管理界面；

![Arm License Management Utility](/img/MDK-Community_Edition/uvision-license-management-utility.c01f97a1de75.png)



3）在***Arm License Management Utility***管理界面中，选择右上角的***License Server***；

4）在***Enter your license server address:***一栏中输入：***https://mdk-preview.keil.arm.com***，并点击***Query***

5）在***Select the product to active:***一栏中，选择***Keil MDK Community (non-commercial free of charge)***，并点击***Active**；

![Arm License Management Utility](/img/MDK-Community_Edition/uvision-license-management-activate.920c19442abe.png)



6）经以上步骤后，若一切顺利的话，您将看到类似以下激活成功的界面：

![Arm License Management Utility](/img/MDK-Community_Edition/uvision-license-management-activated.2cf79d8971e4.png)



## 4、可能出现的问题

若您的电脑网络出现故障，可能步骤4点击完 "Query" 后会出现以下状况：

1）长时间未出现 "Select the product to active:" 一栏

2）出现红色字样提示 "License server 'https://mdk-preview.keil.arm.com' could not be reached. Please verify the license server address and check your network connectivity. Also ensure this license server is for user-based licensing technology. More information on the different licensing technologies is available at https://lm.arm.com."

请排查您的网络，并重新点击 "Query" 或者在浏览器中输入网址 "https://mdk-preview.keil.arm.com" 确认网络是否正常。

