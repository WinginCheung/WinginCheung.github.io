---
layout:     post
title:      Gitbook踩坑记录
subtitle:   Gitbook问题处理记录
date:       2024-10-16
author:     Wingin Cheung
header-img: img/gitbook.avif
catalog: true
mermaid: true
tags:
    - gitbook


---

# Gitbook踩坑记录

测试环境：

+   Ubuntu 24.04.1 LTS



Gitbook是个挺好用的Node.js应用，但是，使用过程中，出现了一些问题，本文仅作记录～

## 1、Gitbook初始化报错

**版本：**

+   GitBook 3.2.3
+   Node.js v18.19.1

**报错信息：**

```shell
$ gitbook init
Installing GitBook 3.2.3
/usr/local/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js:287
      if (cb) cb.apply(this, arguments)
                 ^

TypeError: cb.apply is not a function
    at /usr/local/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js:287:18
    at FSReqCallback.oncomplete (node:fs:203:5)

Node.js v18.19.1
```

**问题根因**：

GitBook与Node.js版本不匹配！

**问题处理**：

安装指定的Node.js v10.24.1版本即可

参考[下载 Node.js®](https://nodejs.org/zh-cn/download/package-manager)：

1）安装 nvm (Node 版本管理器)：

```shell
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
```

***不要使用root或者其他用户进行该操作，因为该脚本需要将nvm下载到`$HOME/.nvm`目录中（例如root用户为`/root/.nvm`）并执行后续操作，可能会导致无法export nvm相关环境变量***

2）重开一个terminal或者输入以下命令手动配置nvm相关环境变量：

```shell
$ export NVM_DIR="$HOME/.nvm"
$ [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
$ [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

3）下载并安装 Node.js（可能需要重启终端）

```shell
$ nvm install 10
```

4）验证环境中是否存在正确的 Node.js 版本v10.24.1

```shell
$ node -v
v10.24.1
```

5）验证环境中是否存在正确的 npm 版本6.14.12

```shell
$ npm -v
6.14.12
```
