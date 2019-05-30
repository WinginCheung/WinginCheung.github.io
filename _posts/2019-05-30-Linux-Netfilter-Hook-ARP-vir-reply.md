---
layout:     post
title:      Linux Module之Netfilter hook（二）
subtitle:   ARP抓包与虚拟回复
date:       2019-05-30
author:     Wingin Cheung
header-img: img/arp-head.jpg
catalog: true
mermaid: true
tags:
    - netfilter
    - hook
    - linux
    - module
    - arp

---

# Linux Module之Netfilter hook -- ARP抓包、虚拟回复

## 1、ARP概述

网络层以上的**协议用IP地址来标识网络接口**，但以太数据帧传输时，**以物理地址来标识网络接口**。因此我们需要进行**IP地址与物理地址**之间的转化。对于**IPv4来说，我们使用ARP地址解析协议**来完成IP地址与物理地址的转化（IPv6使用邻居发现协议进行IP地址与物理地址的转化，它包含在ICMPv6中）。 ARP协议提供了网络层地址（IP地址）到物理地址（mac地址）之间的动态映射。**ARP协议是地址解析的通用协议**。

以下是以太网的arp数据包结构：

<table border="1" align="center">
  <tr>
    <th>ether_dhost</th>
    <th>ether_shost</th>
    <th>ether_type</th>
    <th>ar_hrd</th>
    <th>ar_pro</th>
    <th>ar_hln</th>
    <th>ar_pln</th>
    <th>ar_op</th>
    <th>arp_sha</th>
    <th>arp_spa</th>
    <th>arp_tha</th>
    <th>arp_tpa</th>
  </tr>
  <tr>
    <th><center>6</center></th>
    <th><center>6</center></th>
    <th><center>2</center></th>
    <th><center>2</center></th>
    <th><center>2</center></th>
    <th><center>1</center></th>
    <th><center>1</center></th>
    <th><center>2</center></th>
    <th><center>6</center></th>
    <th><center>4</center></th>
    <th><center>6</center></th>
    <th><center>4</center></th>
  </tr>
  <tr>
    <th colspan="3" rowspan="2"><center><middle>ether_header</middle></center></th>
    <th colspan="5"><center>arphdr</center></th>
  </tr>
  <tr>
    <th colspan="9"><center>ether_arp</center></th>
  </tr>
</table>
 其中，

- ether_type为以太网帧类型，常见类型如下：
    - 0x0800：IPv4数据包；
    - 0x0806：ARP数据包；
    - 0x8035：RAPP数据包；
- ar_hrd为arp硬件类型，如值为1则表示以太网地址；
- ar_pro为arp协议类型，如0x0800表示IPv4；
- ar_op为arp消息类型：
    - 1：ARP Request
    - 2：ARP Reply
    - 3：RARP Request
    - 4：RARP Reply

## 2、相关结构体

### 2.1 ether_header

以太网首部结构体定义于net/ethernet.h中：

```c
struct ether_header {
    u_int8_t ether_dhost[ETH_ALEN];    /* destination eth addr */
    u_int8_t ether_shost[ETH_ALEN];    /* source ether addr */
    u_int16_t ether_type;              /* packet type ID field */
}__attribute__((__packed__));
```

### 2.2 arphdr

arp首部结构体定义于net/if_arp.h中：

```c
struct arphdr {
    u_int16_t ar_hrd;    /* Format of hardware address */
    u_int16_t ar_pro;    /* Format of protocol address */
    u_int8_t ar_hln;     /* Length of hardware address */
    u_int8_t ar_pln;     /* Length of protocol address */
    u_int16_t ar_op;     /* ARP opcode ( command ) */
}
```

### 2.3 ether_arp

arp包结构体定义于netinet/if_ether.h中：

```c
struct ether_arp {
    struct arphdr ea_hdr;          /* fixed-size header */
    u_int8_t arp_sha[ETH_ALEN];    /* sender hardware address */
    __be32 arp_spa;                /* sender protocol address */
    u_int8_t arp_tha[ETH_ALEN];    /* target hardware address */
    __be32 arp_tpa;                /* target protocol address */
};

#define arp_hrd ea_hdr.ar_hrd

#define arp_pro ea_hdr.ar_pro

#define arp_hln ea_hdr.ar_hln

#define arp_pln ea_hdr.ar_pln

#define arp_op  ea_hdr.ar_op
```

## 3、用netfilter抓取ARP数据包

好了，arp相关知识我们已经了解了，我们来用netfilter hook抓取arp数据包来加深一下了解。

首先，我们先搭个程序框架，方便之后在这框架中添加东西：

```c
/* module */
#include <linux/kernel.h>

#include <linux/module.h>

/* network */
#include <linux/if.h>

#include <linux/in.h>

#include <linux/ip.h>

#include <linux/init.h>

#include <linux/inet.h>

#include <linux/skbuff.h>

#include <linux/if_ether.h>

#include <linux/netdevice.h>

#include <linux/inetdevice.h>

#include <linux/if_packet.h>

#include <linux/netfilter.h>

#include <linux/netfilter_bridge.h>

#include <linux/netfilter_arp.h>

#include <linux/if_arp.h>


#ifndef IP_ALEN

#define IP_ALEN 4

#endif


#define PRINT_INFO( s )    printk(KERNEL_INFO ( s ))


static unsigned int arp_input_hook_func(unsigned int hooknum, \
                                        struct sk_buff *skb, \
                                        const struct net_device *in, \
                                        const struct net_device *out, \
                                        int (*okfn)(struct sk_buff *))
{
    return NF_ACCEPT;
}

static struct nf_hook_ops netfileter_hook_ops[] = {
    {
        .hook = arp_input_hook_func,
        .pf = NFPROTO_ARP,
        .hooknum = NF_ARP_IN,
        .priority = NF_IP_PRI_FIRST,
    },
    {},
};

int init_module( void )
{
    if( nf_register_hooks(netfilter_hook_ops, ARRAY_SIZE( netfileter_hook_ops ))){
        PRINT_INFO("%s: nf_register_hooks() failed.\n", __FUNCTION__);
        return -1;
    }

    PRINT_INFO("netfilter hook initilize success.(version 1.0)\n");
    return 0;
}

void cleanup_module( void )
{
    nf_unregister_hooks(netfilter_hook_ops, ARRAY_SIZE( netfilter_hook_ops ));
    PRINT_INFO("netfilter hook have been quit.\n");
    return ;
}

MODULE_LICENSE( "GPL" );
MODULE_AUTHOR( "Wingin Cheung" );
MODULE_DESCRIPTION( "vir-reply for arp in the module." );
```

酱紫程序框架就搭好了～

我们定义个结构体来缓存我们接收到的数据包相关数据，顺便写些函数打印一下：

```c
struct etherpacket_info {
    int dstflag;
    __be32 src_ipaddr;
    __be32 dst_ipaddr;
    u_int8_t src_hwaddr[ETH_ALEN];
    u_int8_t dst_hwaddr[ETH_ALEN];
};

enum print_type {
    Print_IPAddr,
    Print_HWAddr,
};

static void print_data(const char *s, int len, enum print_type type)
{
    int i = 0;
    char *p =(char *)s;

    for(i = 0; i < len; i++ ){
        PRINT_INFO((type == Print_IPAddr)? "%d":"%02X", *p++);
        if( --count ){
            PRINT_INFO((type == Print_IPAddr)? ".":":");
        }
    }
}

static void print_etherpacket_info(const struct etherpacket_info *pkt)
{
    PRINT_INFO("\n----------------ether packet information----------------");
    PRINT_INFO("\nsrc hwaddr: ");
    print_data( pkt->src_hwaddr, ETH_ALEN, Print_HWAddr);
    PRINT_INFO("\nsrc ipaddr: ");
    print_data( pkt->src_ipaddr, IP_ALEN, Print_IPAddr);
    PRINT_INFO("\ndst hwaddr: ");
    print_data( pkt->dst_hwaddr, ETH_ALEN, Print_HWAddr);
    PRINT_INFO("\ndst ipaddr: ");
    print_data( pkt->dst_ipaddr, IP_ALEN, Print_IPAddr);
    PRINT_INFO("\n-------------------------------------------------------\n");
}

static int initialize_ethpkt(struct etherpacket_info *pkt)
{
    pkt =(struct etherpacket_info *)kmalloc( sizeof( struct etherpacket_info ), GFP_KERNEL);
    if(pkt == NULL){
        printk(KERN_ERR "%s: kmalloc error.\n", __FUNCTION__);
    } else {
        memset((void *)pkt, 0x00, sizeof( struct etherpacket_info ));
    }
    
    return pkt;
}

static void clean_ethpkt(struct etherpacket_info *pkt)
{
    if(pkt == NULL){
        return;
    }
    
    kfree( pkt );
    pkt = NULL;
}
```

Ok，现在我们来完善一下相关程序，抓取数据包、然后解析相关字段，再打印出来：

```c
static struct etherpacket_info *ethpkt = NULL;

static unsigned int arp_input_hook_func(unsigned int hooknum, \
                                        struct sk_buff *skb, \
                                        const struct net_device *in, \
                                        const struct net_device *out, \
                                        int (*okfn)(struct sk_buff *))
{
    if( unlikely( !skb )){
        return NF_ACCEPT;
    }
    
    struct arphdr *arph = arp_hdr( skb );
    if( unlikely( !arph )){
        return NF_ACCEPT;
    }
    
    struct ether_arp *etharp =(struct ether_arp *)arph;
    memcpy((void *)&etharp->src_hwaddr,(const void *)&etharp->arp_sha, ETH_ALEN);
    memcpy((void *)&etharp->dst_hwaddr,(const void *)&etharp->arp_tha, ETH_ALEN);
    ethpkt->src_ipaddr = etharp->arp_spa;
    ethpkt->dst_ipaddr = etharp->arp_tpa;
    
    print_etherpacket_info( ethpkt );
    
    return NF_ACCEPT;
}

int init_module( void )
{
    if( initialize_ethpkt( ethpkt )== NULL ){
        return -1;
    }
    
    if( nf_register_hooks(netfilter_hook_ops, ARRAY_SIZE( netfileter_hook_ops ))){
        PRINT_INFO("%s: nf_register_hooks() failed.\n", __FUNCTION__);
        return -1;
    }

    PRINT_INFO("netfilter hook initilize success.(version 1.0)\n");
    return 0;
}

void cleanup_module( void )
{
    clean_ethpkt( ethpkt );
    nf_unregister_hooks(netfilter_hook_ops, ARRAY_SIZE( netfilter_hook_ops ));
    PRINT_INFO("netfilter hook have been quit.\n");
    return ;
}
```

通过以上操作，我们已经实现ARP数据包的抓取和解析工作，打印信息可以通过在终端中输入dmesg查看。

## 4、在netfilter中回复ARP数据包

如果接收到的arp数据包是发往本机的，那我们直接返回NF_ACCEPT，交由系统去处理。

为了判断是否是发往本机的数据吧，我们提前采集相关网卡相关信息。假设我们本机有两个网卡，分别为eth0、eth1，我们可以通过网卡名称来采集：

```c
struct localinterface_info {
    char *name;
    int mtu;
    __be32 mask;
    __be32 ipaddr;
    __be32 brdaddr;
    char hwaddr[ETH_ALEN];
    bool linkstate;
};

#define LOCAL_INTERFACE_NUM    2
static const char eth0name[] = "eth0";
static const char eth1name[] = "eth1";

enum interface_num {
    eth0,
    eth1,
};

static struct localinterface_info LocalInterfaceInfo[LOCAL_INTERFACE_NUM];

static int get_local_interface_info( void )
{
    int i = 0;
    
    for(i = 0; i < LOCAL_INTERFACE_NUM; i++ ){
        struct net_device *dev = dev_get_by_name(&init_net, LocalInterfaceInfo[i].name );
        memcpy((void *)LocalInterfaceInfo[i].hwaddr, dev->dev_addr, ETH_ALEN);
        LocalInterfaceInfo[i].mtu = dev->mtu;
        
        struct in_device *ind = in_dev_get( dev );
        if( ind ){
            struct in_ifaddr *ina =(struct in_ifaddr *)ind->ifa_list;
            if( ina ){
                LocalInterfaceInfo[i].ipaddr = ina->ifa_address;
                LocalInterfaceInfo[i].brdaddr = ina->ifa_broadcast;
                LocalInterfaceInfo[i].mask = ina->ifa_mask;
            }
        }

        dev_put( dev );
    }
    
    return 0;
}

static int initialize_local_interface_info( void )
{
    LocalInterfaceInfo[eth0].name = eth0name;
    LocalInterfaceInfo[eth1].name = eth1name;
    get_local_interface_info();
    
    return 0;
}
```

网卡信息采集完成后，我们可以在接收到arp数据包后对于数据包相关信息和网卡信息，判断是否需要将数据交由系统去处理。

```c
/* base on enum interface_num{} */
enum packet_dst {
    FromEth0,
    FromEth1,
    ToLocal,
};

static void GetPacketDst(struct etherpack_info *pkt)
{
    int i = 0;
    
    for(i = 0; i < LOCAL_INTERFACE_NUM; i++ ){
        if(pkt->dst_ipaddr == LocalInterfaceInfo[i].ipaddr){     /* the packet is send to local-network */
            break;
        } else if((pkt->src_ipaddr & LocalInterfaceInfo[i].mask)== \
                  (LocalInterfaceInfo[i].ipaddr & LocalInterfaceInfo[i].mask)){
            pkt->dstflag = i;
            break;
        }
    }
    
    pkt->dstflag = ToLocal;
}

static unsigned int arp_input_hook_func(unsigned int hooknum, \
                                        struct sk_buff *skb, \
                                        const struct net_device *in, \
                                        const struct net_device *out, \
                                        int (*okfn)(struct sk_buff *))
{
    if( unlikely( !skb )){
        return NF_ACCEPT;
    }
    
    struct arphdr *arph = arp_hdr( skb );
    if( unlikely( !arph )){
        return NF_ACCEPT;
    }
    
    struct ether_arp *etharp =(struct ether_arp *)arph;
    memcpy((void *)&etharp->src_hwaddr,(const void *)&etharp->arp_sha, ETH_ALEN);
    memcpy((void *)&etharp->dst_hwaddr,(const void *)&etharp->arp_tha, ETH_ALEN);
    ethpkt->src_ipaddr = etharp->arp_spa;
    ethpkt->dst_ipaddr = etharp->arp_tpa;
    
    print_etherpacket_info( ethpkt );
    
    GetPacketDst( ethpkt );
    if(ethpkt->dstflag == ToLocal){
        return NF_ACCEPT;
    }
    
    /* the packet is not to send to local-network, then forward or vir-reply in here */
    
    return NF_DROP;
}
```

现在我们来对非本机的arp数据包进行处理。例如，我们制定两个规则：

+ ***非本地数据包，根据源IP地址判断它哪里来、请它哪里回去***
+ ***将目的IP地址的mac地址设置为所进来网卡的mac地址***

例如，目前我们网卡相关信息为：

+ eth0的IP地址为192.168.1.100，mac地址为：3E:B2:88:59:E2:32；

+ eth1的IP地址为192.168.2.200，mac地址为：E3:AB:78:GD:98:43；

现在，我们接收到的数据包信息为：

+ 源IP地址为：192.168.2.150，源mac地址为：AC:0E:AD:CB:04:E2；
+ 目标IP地址为：192.168.2.170，目标mac地址为：FF:FF:FF:FF:FF:FF；

哈？你问我为什么目标mac地址为FF:FF:FF:FF:FF:FF？

这个……

真的难倒我了……

都有mac地址了，源机器还要掂着IP地址这个"人命"，还要到处问目标机器的mac地址这个"门牌号”么？4不4傻了呢？



该数据包的源IP地址与eth1匹配，假设它从eth1来，回复的数据包中，将目标mac地址修改为eth1的mac地址，即为E3:AB:78:GD:98:43。

好了，让我们开始按这规则处理数据包吧：

```c
static unsigned int arp_input_hook_func(unsigned int hooknum, \
                                        struct sk_buff *skb, \
                                        const struct net_device *in, \
                                        const struct net_device *out, \
                                        int (*okfn)(struct sk_buff *))
{
    ...
    
    GetPacketDst( ethpkt );
    if(ethpkt->dstflag == ToLocal){
        return NF_ACCEPT;
    }
    
    memcpy((void *)&etharp->dst_hwaddr,(const char *)LocalInterfaceInfo[i].hwaddr, ETH_ALEN);
    arp_send(ARPOP_REPLY, ETH_P_ARP, ethpkt->src_ipaddr, skb->dev, \
             ethpkt->dst_ipaddr, ethpkt->src_hwaddr, \
             ethpkt->dst_hwaddr, ethpkt->src_hwaddr);
    
    return NF_DROP;
}
```

嗯哼？好像少了arp_send()函数？

莫慌，发挥一下我们程序yuan懒到要命的精神，我们从Linux源码中拷贝一份来用，各种宏定义我都懒得理它做什么的了(o^^o)：

```c
static struct sk_buff *arp_create(int type, int ptype, __be32 dest_ip, \
                                  struct net_device *dev, __be32 src_ip, \
                                  const unsigned char *dest_hw, \
                                  const unsigned char *src_hw, \
                                  const unsigned char *target_hw)
{
    struct sk_buff *skb;
    struct arphdr *arp;
    unsigned char *arp_ptr;
    int hlen = LL_RESERVED_SPACE(dev);
    int tlen = dev->needed_tailroom;

    skb = alloc_skb(arp_hdr_len(dev) + hlen + tlen, GFP_ATOMIC);
    if (skb == NULL)
        return NULL;

    skb_reserve(skb, hlen);
    skb_reset_network_header(skb);
    arp = (struct arphdr *) skb_put(skb, arp_hdr_len(dev));
    skb->dev = dev;
    skb->protocol = htons(ETH_P_ARP);
    if (src_hw == NULL)
        src_hw = dev->dev_addr;
    if (dest_hw == NULL)
        dest_hw = dev->broadcast;

    if (dev_hard_header(skb, dev, ptype, dest_hw, src_hw, skb->len) < 0)
        goto out;

    switch (dev->type) {
#if IS_ENABLED(CONFIG_AX25)
        case ARPHRD_AX25:
            arp->ar_hrd = htons(ARPHRD_AX25);
            arp->ar_pro = htons(AX25_P_IP);
            break;

#if IS_ENABLED(CONFIG_NETROM)
        case ARPHRD_NETROM:
            arp->ar_hrd = htons(ARPHRD_NETROM);
            arp->ar_pro = htons(AX25_P_IP);
            break;
#endif
#endif

#if IS_ENABLED(CONFIG_FDDI)
        case ARPHRD_FDDI:
            arp->ar_hrd = htons(ARPHRD_ETHER);
            arp->ar_pro = htons(ETH_P_IP);
            break;
#endif
        default:
            arp->ar_hrd = htons(dev->type);
            arp->ar_pro = htons(ETH_P_IP);
            break;
    }

    arp->ar_hln = dev->addr_len;
    arp->ar_pln = 4;
    arp->ar_op = htons(type);

    arp_ptr = (unsigned char *)(arp + 1);

    memcpy(arp_ptr, src_hw, dev->addr_len);
    arp_ptr += dev->addr_len;
    memcpy(arp_ptr, &src_ip, 4);
    arp_ptr += 4;

    switch (dev->type) {
#if IS_ENABLED(CONFIG_FIREWIRE_NET)
        case ARPHRD_IEEE1394:
            break;
#endif
        default:
            if (target_hw != NULL)
                memcpy(arp_ptr, target_hw, dev->addr_len);
            else
                memcpy(arp_ptr, 0, dev->addr_len);
            arp_ptr += dev->addr_len;
    }
    memcpy(arp_ptr, &dest_ip, 4);

    return skb;

out:
    kfree_skb(skb);
    return NULL;
}

static void arp_xmit(struct sk_buff *skb)
{
    NF_HOOK(NFPROTO_ARP, NF_ARP_OUT, skb, NULL, skb->dev, dev_queue_xmit);
}

static void arp_send(int type, int ptype, __be32 dest_ip, \
                     struct net_device *dev, __be32 src_ip, \
                     const unsigned char *dest_hw, \
                     const unsigned char *src_hw, \
                     const unsigned char *target_hw)
{
    struct sk_buff *skb;

    if (dev->flags&IFF_NOARP)
        return;

    skb = arp_create(type, ptype, dest_ip, dev, src_ip, \
                     dest_hw, src_hw, target_hw);
    if (skb == NULL)
        return;

    arp_xmit(skb);
}
```

好了，现在全齐了，没什么了吧？

## 5、程序终结者

整理一下程序，来个"整整齐齐的一家人"：

```c
/* file name: netfilter_arp_hook.c */

/* module */
#include <linux/kernel.h>

#include <linux/module.h>

/* network */
#include <linux/if.h>

#include <linux/in.h>

#include <linux/ip.h>

#include <linux/init.h>

#include <linux/inet.h>

#include <linux/skbuff.h>

#include <linux/if_ether.h>

#include <linux/netdevice.h>

#include <linux/inetdevice.h>

#include <linux/if_packet.h>

#include <linux/netfilter.h>

#include <linux/netfilter_bridge.h>

#include <linux/netfilter_arp.h>

#include <linux/if_arp.h>


#ifndef IP_ALEN

#define IP_ALEN 4

#endif


#define PRINT_INFO( s )    printk(KERNEL_INFO ( s ))


struct etherpacket_info {
    int dstflag;
    __be32 src_ipaddr;
    __be32 dst_ipaddr;
    u_int8_t src_hwaddr[ETH_ALEN];
    u_int8_t dst_hwaddr[ETH_ALEN];
};

enum print_type {
    Print_IPAddr,
    Print_HWAddr,
};

struct localinterface_info {
    char *name;
    int mtu;
    __be32 mask;
    __be32 ipaddr;
    __be32 brdaddr;
    char hwaddr[ETH_ALEN];
    bool linkstate;
};

#define LOCAL_INTERFACE_NUM    2

static const char eth0name[] = "eth0";
static const char eth1name[] = "eth1";

enum interface_num {
    eth0,
    eth1,
};

/* base on enum interface_num{} */
enum packet_dst {
    FromEth0,
    FromEth1,
    ToLocal,
};

static struct etherpacket_info *ethpkt = NULL;
static struct localinterface_info LocalInterfaceInfo[LOCAL_INTERFACE_NUM];

static void print_data(const char *s, int len, enum print_type type)
{
    int i = 0;
    char *p =(char *)s;

    for(i = 0; i < len; i++ ){
        PRINT_INFO((type == Print_IPAddr)? "%d":"%02X", *p++);
        if( --count ){
            PRINT_INFO((type == Print_IPAddr)? ".":":");
        }
    }
}

static void print_etherpacket_info(const struct etherpacket_info *pkt)
{
    PRINT_INFO("\n----------------ether packet information----------------");
    PRINT_INFO("\nsrc hwaddr: ");
    print_data( pkt->src_hwaddr, ETH_ALEN, Print_HWAddr);
    PRINT_INFO("\nsrc ipaddr: ");
    print_data( pkt->src_ipaddr, IP_ALEN, Print_IPAddr);
    PRINT_INFO("\ndst hwaddr: ");
    print_data( pkt->dst_hwaddr, ETH_ALEN, Print_HWAddr);
    PRINT_INFO("\ndst ipaddr: ");
    print_data( pkt->dst_ipaddr, IP_ALEN, Print_IPAddr);
    PRINT_INFO("\n-------------------------------------------------------\n");
}

static int initialize_ethpkt(struct etherpacket_info *pkt)
{
    pkt =(struct etherpacket_info *)kmalloc( sizeof( struct etherpacket_info ), GFP_KERNEL);
    if(pkt == NULL){
        printk(KERN_ERR "%s: kmalloc error.\n", __FUNCTION__);
    } else {
        memset((void *)pkt, 0x00, sizeof( struct etherpacket_info ));
    }
    
    return pkt;
}

static void clean_ethpkt(struct etherpacket_info *pkt)
{
    if(pkt == NULL){
        return;
    }
    
    kfree( pkt );
    pkt = NULL;
}

static int get_local_interface_info( void )
{
    int i = 0;
    
    for(i = 0; i < LOCAL_INTERFACE_NUM; i++ ){
        struct net_device *dev = dev_get_by_name(&init_net, LocalInterfaceInfo[i].name );
        memcpy((void *)LocalInterfaceInfo[i].hwaddr, dev->dev_addr, ETH_ALEN);
        LocalInterfaceInfo[i].mtu = dev->mtu;
        
        struct in_device *ind = in_dev_get( dev );
        if( ind ){
            struct in_ifaddr *ina =(struct in_ifaddr *)ind->ifa_list;
            if( ina ){
                LocalInterfaceInfo[i].ipaddr = ina->ifa_address;
                LocalInterfaceInfo[i].brdaddr = ina->ifa_broadcast;
                LocalInterfaceInfo[i].mask = ina->ifa_mask;
            }
        }

        dev_put( dev );
    }
    
    return 0;
}

static int initialize_local_interface_info( void )
{
    LocalInterfaceInfo[eth0].name = eth0name;
    LocalInterfaceInfo[eth1].name = eth1name;
    get_local_interface_info();
    
    return 0;
}

static void GetPacketDst(struct etherpack_info *pkt)
{
    int i = 0;
    
    for(i = 0; i < LOCAL_INTERFACE_NUM; i++ ){
        if(pkt->dst_ipaddr == LocalInterfaceInfo[i].ipaddr){     /* the packet is send to local-network */
            break;
        } else if((pkt->src_ipaddr & LocalInterfaceInfo[i].mask)== \
                  (LocalInterfaceInfo[i].ipaddr & LocalInterfaceInfo[i].mask)){
            pkt->dstflag = i;
            break;
        }
    }
    
    pkt->dstflag = ToLocal;
}

static struct sk_buff *arp_create(int type, int ptype, __be32 dest_ip, \
                                  struct net_device *dev, __be32 src_ip, \
                                  const unsigned char *dest_hw, \
                                  const unsigned char *src_hw, \
                                  const unsigned char *target_hw)
{
    struct sk_buff *skb;
    struct arphdr *arp;
    unsigned char *arp_ptr;
    int hlen = LL_RESERVED_SPACE(dev);
    int tlen = dev->needed_tailroom;

    skb = alloc_skb(arp_hdr_len(dev) + hlen + tlen, GFP_ATOMIC);
    if (skb == NULL)
        return NULL;

    skb_reserve(skb, hlen);
    skb_reset_network_header(skb);
    arp = (struct arphdr *) skb_put(skb, arp_hdr_len(dev));
    skb->dev = dev;
    skb->protocol = htons(ETH_P_ARP);
    if (src_hw == NULL)
        src_hw = dev->dev_addr;
    if (dest_hw == NULL)
        dest_hw = dev->broadcast;

    if (dev_hard_header(skb, dev, ptype, dest_hw, src_hw, skb->len) < 0)
        goto out;

    switch (dev->type) {
#if IS_ENABLED(CONFIG_AX25)
        case ARPHRD_AX25:
            arp->ar_hrd = htons(ARPHRD_AX25);
            arp->ar_pro = htons(AX25_P_IP);
            break;

#if IS_ENABLED(CONFIG_NETROM)
        case ARPHRD_NETROM:
            arp->ar_hrd = htons(ARPHRD_NETROM);
            arp->ar_pro = htons(AX25_P_IP);
            break;
#endif
#endif

#if IS_ENABLED(CONFIG_FDDI)
        case ARPHRD_FDDI:
            arp->ar_hrd = htons(ARPHRD_ETHER);
            arp->ar_pro = htons(ETH_P_IP);
            break;
#endif
        default:
            arp->ar_hrd = htons(dev->type);
            arp->ar_pro = htons(ETH_P_IP);
            break;
    }

    arp->ar_hln = dev->addr_len;
    arp->ar_pln = 4;
    arp->ar_op = htons(type);

    arp_ptr = (unsigned char *)(arp + 1);

    memcpy(arp_ptr, src_hw, dev->addr_len);
    arp_ptr += dev->addr_len;
    memcpy(arp_ptr, &src_ip, 4);
    arp_ptr += 4;

    switch (dev->type) {
#if IS_ENABLED(CONFIG_FIREWIRE_NET)
        case ARPHRD_IEEE1394:
            break;
#endif
        default:
            if (target_hw != NULL)
                memcpy(arp_ptr, target_hw, dev->addr_len);
            else
                memcpy(arp_ptr, 0, dev->addr_len);
            arp_ptr += dev->addr_len;
    }
    memcpy(arp_ptr, &dest_ip, 4);

    return skb;

out:
    kfree_skb(skb);
    return NULL;
}

static void arp_xmit(struct sk_buff *skb)
{
    NF_HOOK(NFPROTO_ARP, NF_ARP_OUT, skb, NULL, skb->dev, dev_queue_xmit);
}

static void arp_send(int type, int ptype, __be32 dest_ip, \
                     struct net_device *dev, __be32 src_ip, \
                     const unsigned char *dest_hw, \
                     const unsigned char *src_hw, \
                     const unsigned char *target_hw)
{
    struct sk_buff *skb;

    if (dev->flags&IFF_NOARP)
        return;

    skb = arp_create(type, ptype, dest_ip, dev, src_ip, \
                     dest_hw, src_hw, target_hw);
    if (skb == NULL)
        return;

    arp_xmit(skb);
}

static unsigned int arp_input_hook_func(unsigned int hooknum, \
                                        struct sk_buff *skb, \
                                        const struct net_device *in, \
                                        const struct net_device *out, \
                                        int (*okfn)(struct sk_buff *))
{
    if( unlikely( !skb )){
        return NF_ACCEPT;
    }
    
    struct arphdr *arph = arp_hdr( skb );
    if( unlikely( !arph )){
        return NF_ACCEPT;
    }
    
    struct ether_arp *etharp =(struct ether_arp *)arph;
    memcpy((void *)&etharp->src_hwaddr,(const void *)&etharp->arp_sha, ETH_ALEN);
    memcpy((void *)&etharp->dst_hwaddr,(const void *)&etharp->arp_tha, ETH_ALEN);
    ethpkt->src_ipaddr = etharp->arp_spa;
    ethpkt->dst_ipaddr = etharp->arp_tpa;
    
    print_etherpacket_info( ethpkt );
    
    GetPacketDst( ethpkt );
    if(ethpkt->dstflag == ToLocal){
        return NF_ACCEPT;
    }
    
    memcpy((void *)&etharp->dst_hwaddr,(const char *)LocalInterfaceInfo[i].hwaddr, ETH_ALEN);
    arp_send(ARPOP_REPLY, ETH_P_ARP, ethpkt->src_ipaddr, skb->dev, \
             ethpkt->dst_ipaddr, ethpkt->src_hwaddr, \
             ethpkt->dst_hwaddr, ethpkt->src_hwaddr);
    
    return NF_DROP;
}

static struct nf_hook_ops netfileter_hook_ops[] = {
    {
        .hook = arp_input_hook_func,
        .pf = NFPROTO_ARP,
        .hooknum = NF_ARP_IN,
        .priority = NF_IP_PRI_FIRST,
    },
    {},
};

int init_module( void )
{
    if( initialize_ethpkt( ethpkt )== NULL ){
        return -1;
    }
    
    if( nf_register_hooks(netfilter_hook_ops, ARRAY_SIZE( netfileter_hook_ops ))){
        PRINT_INFO("%s: nf_register_hooks() failed.\n", __FUNCTION__);
        return -1;
    }

    PRINT_INFO("netfilter hook initilize success.(version 1.0)\n");
    return 0;
}

void cleanup_module( void )
{
    clean_ethpkt( ethpkt );
    nf_unregister_hooks(netfilter_hook_ops, ARRAY_SIZE( netfilter_hook_ops ));
    PRINT_INFO("netfilter hook have been quit.\n");
    return ;
}

MODULE_LICENSE( "GPL" );
MODULE_AUTHOR( "Wingin Cheung" );
MODULE_DESCRIPTION( "vir-reply for arp in the module." );
```

再来个Makefile：

```Makefile
# Mackfile

obj-M += netfilter_arp_hook.o

all:
	$(MAKE) -C /lib/modules/$(shell uname -r)/buile M=$(PWD) modules

clean:
	$(MAKE) -C /lib/modules/$(shell uname -r)/buile M=$(PWD) clean
```

直接在终端中输入make来编译、insmod来装载吧～

什么？error一堆？系统宕机？尴尬……

那，BTW，

告诉你一个不好的消息，能不能编译通过随缘～

编译通过了系统宕机？那也蛮正常的～

毕竟，上边的一切，都是我编的，真是编、的，不骗你……

有问题，E-mail告诉我吧，当然其它能联系到我的方式也可以～

到时我再修正错误～

Of course，我是打死不承认错误的～

我没错，是你的姿势不对，哈哈～

没错，我就是这么任性、这么皮，就问你还扶墙么？