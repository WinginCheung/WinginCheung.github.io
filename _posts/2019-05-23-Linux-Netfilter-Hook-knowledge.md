---
layout:     post
title:      Linux Module之Netfilter hook（一）
subtitle:   Netfilter hook基础知识
date:       2019-05-23
author:     Wingin Cheung
header-img: img/netfilter-head.jpg
catalog: true
mermaid: true
tags:
    - netfilter
    - hook
    - linux
    - module

---

# Linux Module之Netfilter hook -- 基础知识

## 1. Netfilter的hook点

​    Netfilter是从Linux 2.4开始引入内核的一个子系统，架构就是在整个网络流程的若干位置放置了一些检测点，而在每个检测点上登记了一些处理函数进行处理，如包过滤、NAT、或者用户自定义的功能。

​    IP层的hook点包括：

1. NF_IP_PRE_ROUTING：刚刚进入网络层的数据包通过此点（刚刚进行完版本号，校验和等检测）， 目的地址转换在此点进行；
2. NF_IP_LOCAL_IN：经路由查找后，送往本机的通过此检查点，INPUT包过滤在此点进行；
3. NF_IP_FORWARD：要转发的包通过此检测点，FORWORD包过滤在此点进行；
4. NF_IP_POST_ROUTING：所有马上便要通过网络设备出去的包通过此检测点，内置的源地址转换功能（包括地址伪装）在此点进行；
5. NF_IP_LOCAL_OUT：本机进程发出的包通过此检测点，OUTPUT包过滤在此点进行；

​    这些hook点的详细位置如下（来自https://www.netfilter.org/documentation/HOWTO/netfilter-hacking-HOWTO-3.html)：

```
A Packet Traversing the Netfilter System:

   --->[1]--->[ROUTE]--->[3]--->[4]--->
                 |            ^
                 |            |
                 |         [ROUTE]
                 v            |
                [2]          [5]
                 |            ^
                 |            |
                 v            |
```

​    内核模块可以注册hook去侦听任何hook点。

## 2. Netfilter返回值

​    注册模块还必须指定hook函数的优先级，以便注册后核心网络代码按顺序优先级顺序调用。

​    内核模块在hook函数完成后，可以返回以下5种类型，告诉核心网络代码接下来如何处理数据包：

+ NF_ACCEPT：正常继续执行
+ NF_DROP：丢弃数据包
+ NF_STOLEN：由模块接管数据包，核心网络代码放弃处理，***数据包资源未被释放，数据包及其各自的sk_buff结构仍然有效，只是其所有权转移到了hook函数而已***
+ NF_QUEUE：对数据包进行列队，通常用于用户空间处理
+ NF_REPEAT：重新调用此hook函数

## 3. nf_hook_ops数据结构体

​    nf_hook_ops数据结构在linux/netfilter.h中定义，该数据结构的定义如下：

```c
struct nf_hook_ops {
    struct list_head list;

    /* User fills in from here down. */
    nf_hookfn *hook;
    int pf;
    int hooknum;
    /* Hooks are ordered in ascending priority. */
    int priority;
};
```

​    其中，

   + list：维护Netfilter hook的列表，注册hook时用户无需关心；
   + hook：hook被调用时执行的函数，是一个指向nf_hookfn类型的函数的指针，详见[4. nf_hookfn函数的原型](#4. nf_hookfn函数的原型)；
   + pf：指定协议簇，有效值在linux/socket.h中列出。例如IPv4使用PF_INET；
   + hooknum：表示这个钩子在数据包处理流程中的位置，即[hook点](#1. Netfilter的hook点)；
   + priority：你所注册的钩子函数优先级。对于IPv4，可接受的值在linux/netfilter_ipv4.h中的nf_ip_hook_priorities枚举中定义，如NF_IP_PRI_FIRST表示优先级最高；

## 4. nf_hookfn函数的原型

​    nf_hookfn函数的原型在linux / netfilter.h中给出如下：

```c
typedef unsigned int nf_hookfn(unsigned int hooknum, \
                                struct sk_buff **skb, \
                                const struct net_device *in, \
                                const struct net_device *out, \
                                int (*okfn)(struct sk_buff *));
```

​    其中，

+ hooknum：为[nf_hook_ops数据结构体](#3. nf_hook_ops数据结构体)中的[hooknum](#1. Netfilter的hook点)；
+ skb：指向[sk_buff结构](#5. sk_buff结构)的指针；
+ in：用于描述数据包到达的接口；
+ out：结构描述了数据包离开的接口；
+ okfn：函数指针，将[sk_buff结构](#5. sk_buff结构)作为唯一参数并返回一个整数；

    ***Tips：skb之后的两个参数是指向net_device结构的指针。net_device结构是Linux内核用来描述各种网络接口的结构。这些结构中的第一个，in，用于描述数据包到达的接口；out结构描述了数据包离开的接口。重要的是，通常只提供这些结构中的一种。例如，只会为NF_IP_PRE_ROUTING和NF_IP_LOCAL_IN挂钩提供 in。 仅为NF_IP_LOCAL_OUT和NF_IP_POST_ROUTING挂钩提供 out。***

## 5. sk_buff结构

​    sk_buff，为socket buffers，简写skb，译为套接字缓存。它作为网络数据包的存放地点，使得协议栈中每个层都可以对数据进行操作，从而实现了数据包自底向上的传递。该结构维护收到的或者要发送的网络包。sk_buff是一个双向链表。

​    sk_buff说明了如何访问存储区空间，如何维护多个存储区空间以及存储网络包解析的结果。内核中sk_buff结构体在各层协议之间传输不是用拷贝sk_buff结构体，而是通过增加协议头和移动指针来操作的。如果是从L4传输到L2，则是通过往sk_buff结构体中增加该层协议头来操作；如果是从L4到L2，则是通过移动sk_buff结构体中的data指针来实现，不会删除各层协议头。

​    sk_buff结构在linux/skbuff.h中定义如下：

```c
struct sk_buff {
    /* These two members must be first. */
    struct sk_buff *next;    /* 因为sk_buff结构体是双链表，所以有前驱后继。这是个指向后面的sk_buff结构体指 */
    struct sk_buff *prev;    /* 这是指向前一个sk_buff结构体指针 */
    /* 老版本（2.6以前）应该还有个字段： sk_buff_head *list, 即每个sk_buff结构都有个指针指向头节点 */
    struct sock *sk;         /* 指向拥有此缓冲的套接字sock结构体，即：宿主传输控制模块 */
    ktime_t tstamp;          /* 时间戳，表示这个skb的接收到的时间，一般是在包从驱动中往二层发送的接口函数中设置 */
    struct net_device *dev;  /* 表示一个网络设备，当skb为输出/输入时，dev表示要输出/输入到的设备 */
    unsigned long _skb_dst;  /* 主要用于路由子系统，保存路由有关的东西 */
    char cb[48];             /* 保存每层的控制信息,每一层的私有信息 */
    unsigned int len,        /* 表示数据区的长度(tail - data)与分片结构体数据区的长度之和。其实这个len中数据区长度是个有效长度,
                              * 因为不删除协议头，所以只计算有效协议头和包内容。如：当在L3时，不会计算L2的协议头长度。
                             */
                 data_len;   /* 只表示分片结构体数据区的长度，所以len = (tail - data) + data_len；*/
    __u16 mac_len,           /* mac报头的长度 */
          hdr_len;           /* 用于clone时，表示clone的skb的头长度 */
    __u32 priority;          /* 优先级，主要用于QOS */
    kmemcheck_bitfield_begin(flags1);
    __u8 local_df:1,         /* 是否可以本地切片的标志 */
         cloned:1,           /* 为1表示该结构被克隆，或者自己是个克隆的结构体；同理被克隆时，自身skb和克隆skb的cloned都要置1 */
         ip_summed:2, 
         nohdr:1,            /* nohdr标识payload是否被单独引用，不存在协议首部. */
                             /* 如果被引用，则决不能再修改协议首部，也不能通过skb->data来访问协议首部. */
         nfctinfo:3;
    __u8 pkt_type:3,         /* 标记帧的类型 */
         fclone:2,           /* 这个成员字段是克隆时使用，表示克隆状态 */
         ipvs_property:1,
         peeked:1,
         nf_trace:1;
    __be16 protocol:16;      /* 这是包的协议类型，标识是IP包还是ARP包或者其他数据包。*/
    kmemcheck_bitfield_end(flags1);
    void (*destructor)(struct sk_buff *skb);  /* 这是析构函数，后期在skb内存销毁时会用到 */
#if( defined(CONFIG_NF_CONNTRACK) || defined(CONFIG_NF_CONNTRACK_MODULE))
  
    struct nf_conntrack	*nfct;
    struct sk_buff *nfct_reasm;
#endif

#ifdef CONFIG_BRIDGE_NETFILTER
  
    struct nf_bridge_info *nf_bridge;
#endif
  
    int iif;                /* 接受设备的index */
#ifdef CONFIG_NET_SCHED
  
    __u16 tc_index;         /* traffic control index */
#ifdef CONFIG_NET_CLS_ACT
  
    __u16 tc_verd;          /* traffic control verdict */
#endif
  
#endif

    kmemcheck_bitfield_begin(flags2);
    __u16 queue_mapping:16;
#ifdef CONFIG_IPV6_NDISC_NODETYPE
  
    __u8 ndisc_nodetype:2;
#endif
  
    kmemcheck_bitfield_end(flags2);
    /* 0/14 bit hole */
#ifdef CONFIG_NET_DMA
  
    dma_cookie_t dma_cookie;
#endif

#ifdef CONFIG_NETWORK_SECMARK

    __u32 secmark;
#endif

    __u32 mark;
    __u16 vlan_tci;
    sk_buff_data_t transport_header;  /* 指向四层帧头结构体指针 */
    sk_buff_data_t network_header;    /* 指向三层IP头结构体指针 */
    sk_buff_data_t mac_header;        /* 指向二层mac头的头 */
    /* These elements must be at the end, see alloc_skb() for details.  */
    sk_buff_data_t tail;              /* 指向数据区中实际数据结束的位置 */
    sk_buff_data_t end;               /* 指向数据区中结束的位置（非实际数据区域结束位置）*/
    unsigned char *head,              /* 指向数据区中开始的位置（非实际数据区域开始位置）*/
                  *data;              /* 指向数据区中实际数据开始的位置 */
    unsigned int truesize;            /* 表示总长度，包括sk_buff自身长度和数据区以及分片结构体的数据区长度 */
    atomic_t users;                   /* skb被克隆引用的次数，在内存申请和克隆时会用到 */
};    /* end of sk_buff */
```

​    其中，

   + char cb[48]：这个字段是skb信息控制块，也就是存储每层的一些协议信息，当数据包在哪一层时，存储的就是哪一层协议信息。这个字段由数据包所在层使用和维护，如果要访问本层协议信息，可以通过用一些宏来操作这个成员字段。如：#define TCP_SKB_CB(__skb)  ((struct tcp_skb_cb *)&((__skb)->cb[0]))；
   + _u8 fclone:2：这是个克隆状态标志，到sk_buff结构内存申请时会使用到。对于fclone值，可为：
     - SKB_FCLONE_UNAVAILABLE：则表明skb未被克隆；
     - SKB_FCLONE_ORIG：则表明是从skbuff_fclone_cache缓存池中分配的父skb，可以被克隆；
     - SKB_FCLONE_CLONE：则表明是在skbuff_fclone_cache分配的子skb，从父skb克隆得到的；
+ atomic_t users：这是个引用计数，表明了有多少实体引用了这个skb。其作用就是在销毁skb结构体时，先查看下users是否为零，若不为零，则调用函数递减下引用计数users即可；当某一次销毁时，users为零才真正释放内存空间。有两个操作函数：atomic_inc()引用计数增加1；atomic_dec()引用计数减去1；
+ unsigned int len：表示当前缓冲区中数据块的大小的总长度。它包括主缓冲中（即是sk_buff结构中指针data指向）的数据区的实际长度（data-tail）和分片中的数据长度。这个长度在数据包在各层间传输时会改变，因为分片数据长度不变，从L2到L4时，则len要减去帧头大小和网络头大小；从L4到L2则相反，要加上帧头和网络头大小。所以：len = (data - tail) + data_len；
+ unsigned int data_len：只计算分片中数据的长度，即是分片结构体中page指向的数据区长度；
+ unsigned int truesize：这是缓冲区的总长度，包括sk_buff结构和数据部分。如果申请一个len字节的缓冲区，alloc_skb函数会把它初始化成len+sizeof(sk_buff)。当skb->len变化时，这个变量也会变化。所以：truesize = len + sizeof(sk_buff) = (data - tail) + data_len + sizeof(sk_buff)；

