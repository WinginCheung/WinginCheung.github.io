---
layout:     post
title:      如何通过修改Hosts文件连接指定网站
subtitle:   如何避免DNS污染等问题
date:       2024-09-27
author:     Wingin Cheung
header-img: img/over_the_walls.png
catalog: true
mermaid: true
tags:
    - Internet


---

# 如何通过修改Hosts文件连接指定网站

测试环境：

+   Ubuntu 24.04.1 LTS



作为在国内的软件工程师、作为一名CV工程师，怎么可能少得了访问那个全球最大的交友网站呢。途径呢？一般我们都是花点小钱钱科学魔法上网。但是，总有那么些不方便嘛，也费钱嘛，所以，为了省钱，我们还是之间修改hosts文件吧。

目标只有一个：获得全球最大交友网站的ip地址，写入到hosts中！

但，哪里获取其ip地址呢？有很多途径，但我喜欢使用 [ipaddress.com](https://www.ipaddress.com/)，在这网站输入对应的网址，即可获取对应的ip地址，然后再拷贝地址到hosts中刷新dns即可。

但，作为菜逼工程师，每次都这么搞、太累了！叔可忍、哥不可忍，脚本搞起来，不解释、不说明、直接拷贝开干！

## Step 1：获取脚本

拷贝以下脚本内容到你喜欢的文本中，例如***update_hosts.sh*** :

```shell
#!/bin/bash

url_ipaddress="https://www.ipaddress.com/website"
url_website="github.com |\
            assets-cdn.github.com |\
            github.global.ssl.fastly.net |\
            raw.githubusercontent.com"

update_hosts() {
    website=$1
    hosts_file=$2
    tempfile="temp.html"
    website_label2="origin"
    website_label2_split_str="<span class=\"$website_label2\"><span>@<\/span><\/span>"

    # get website
    wget -q $url_ipaddress/$website -O $tempfile
    if [ $? -ne 0 ]; then
        echo "get $website's ipaddress error"
        return
    fi

    # get website's label 2
    label2=`echo ${website%.*}`
    if [[ $label2 == *\.* ]]; then
        website_label2=`echo ${label2%.*}`
        website_label2_split_str=$website_label2
    fi

    # get DNS resource records
    records=`cat $tempfile |\
            sed ':a;N;$!ba;s/\n/;/g' |\
            sed "s/Records<\/span><\/h3><pre>$website_label2_split_str\ \ IN\ \ A\ \ <a\ href=\"https:\/\/www.ipaddress.com\/ipv4\//IN_A_START_STR/g" |\
            sed 's/<\/a><\/pre><\/div><div\ id=\"tabpanel-dns-aaaa\"\ role=\"tabpanel/IN_A_STOP_STR/g' |\
            awk -F 'IN_A_START_STR' '{print $2}' |\
            awk -F 'IN_A_STOP_STR' '{print $1}'`

    # only one?
    if [[ $records == *$website_label2* ]]; then
        ip_list=`echo $records |\
            sed 's/<\/a><br\/>//g' |\
            sed "s/$website_label2_split_str\ IN\ A\ <a\ href=\"https:\/\/www.ipaddress.com\/ipv4\//\ /g"`
    else
        ip_list=$records
    fi

    # remove website
    sed -i "/$website/d" $hosts_file

    for ip in $ip_list; do
        ip=`echo $ip | sed 's/.*\">//'`
        echo $ip $website >> $hosts_file
    done

    rm -rf $tempfile
}


[ $UID -eq 0 ] && hosts_file="/etc/hosts" || hosts_file=$(pwd)"/hosts"
echo "config file $hosts_file"

url_website=`echo $url_website | sed 's/|//g'`
for website in $url_website; do
    update_hosts $website $hosts_file
done

# update dns
[ $UID -eq 0 ] && sudo systemctl restart systemd-resolved
```

## Step 2：给脚本加执行权限

```shell
$ chmod +x <shell-name>
```

例如给***update_hosts.sh***加权限：

```shell
$ chmod +x update_hosts.sh
```

## Step 3：执行脚本

### 3.1 普通用户权限下测试脚本

我很菜，所以脚本也可能有问题、你也可能不放心嘛，所以，我们还是先试试更新效果嘛。把hosts文件拷贝到脚本同一目录再执行脚本：

```shell
$ cd <shell-path>
$ cp /etc/hosts .
$ bash ./<shell-name>
```

例如，在脚本***update_hosts.sh***在用户根目录下，就这样玩：

```shell
$ cd ~
$ cp /etc/hosts .
$ bash ./update_hosts.sh
```

现在你可以看到根目录下的hosts文件中是否有github相关配置了：

```shell
$ cat hosts
...
140.82.113.4 github.com
185.199.108.153 assets-cdn.github.com
185.199.109.153 assets-cdn.github.com
185.199.110.153 assets-cdn.github.com
185.199.111.153 assets-cdn.github.com
151.101.1.194 github.global.ssl.fastly.net
151.101.65.194 github.global.ssl.fastly.net
151.101.129.194 github.global.ssl.fastly.net
151.101.193.194 github.global.ssl.fastly.net
185.199.108.133 raw.githubusercontent.com
185.199.109.133 raw.githubusercontent.com
185.199.110.133 raw.githubusercontent.com
185.199.111.133 raw.githubusercontent.com
```

### 3.2 root权限下执行脚本

我们已经测试过脚本符合我们的预期了嘛，所以来梭哈一把嘛，加sudo重新执行！

```shell
$ sudo bash ./update_hosts.sh
```

什么？没看到hosts更新？别开玩笑，这时候更新的不是当前目录下的这个hosts文件，而是***/etc/hosts***！！！！！不信你去看嘛～

嗯？还需不需要刷新dns或重启网络？***脚本里已经执行了刷新dns操作***，你要是不放心，也可以再执行一次的嘛

## Step 4：自定义脚本

什么？油管什么的你也想看？这个……嗯，修改hosts的方式好像不太行的，放弃吧，不行你就魔法上网吧……

但是你想稳定访问一些你想要的网站，可以尝试修改一下脚本，修改哪里呢？在***url_website***字段中增加即可，例如，增加***gitbook.com***：

```diff
url_website="github.com |\
            assets-cdn.github.com |\
            github.global.ssl.fastly.net |\
-            raw.githubusercontent.com"
+            raw.githubusercontent.com |\
+            gitbook.com"
```



嗯，修改hosts不是万能的！墙外很危险，还是不要玩了哈，听话……
