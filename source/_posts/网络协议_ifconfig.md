---
title: ifconfig
date: 2020-03-20
categories:
  - 网络协议
tags:
  - 网络协议
---
> `ip地址`是一个网卡在网络世界的通信地址、类似`门牌号`

![32位ip地址分类.png](https://upload-images.jianshu.io/upload_images/14027542-73bac7cc574fb7e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 无类型域间选路(CIDR)
从上图可以看到、C类地址能包含的主机数只有 2<sup>8</sup> = 254个, 数量过少; 而B类地址能包含的最大主机数量又太多了、一般企业规模达不到65534、剩余的地址就是浪费.

于是产生了一个折中方案: CIDR. 打破ip类别设计、将32位ip一分为二、前边是网络号、后边是主机号.
eg. 10.100.122.2/24  
`/ ` 表示前24位代表网络号、后8位是主机号

伴随CIDR存在的还有`广播地址` 和 `子网掩码`.
广播地址是: 10.100.122.255, 若发送这个地址、所有10.100.122 网络里的机器都可以收到
子网掩码: 255.255.255.0

将子网掩码 和 ip 进行 AND运算、可以得到网络号

#### 公有ip地址 和 私有ip地址
![私有ip地址范围.png](https://upload-images.jianshu.io/upload_images/14027542-6f0cb3fd1a380ca4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`组播地址`: 属于某个组的机器都能收到消息.
eg. 邮件组

在ip地址的后边、有个scope, 对eth0这个网卡来讲、是global, 说明这个网卡是可以对外的、可以接收来自各个地方的包. 对于lo来讲、是host, 说明这张网卡仅可以供本机相互通信.

lo 全程 loopback, 又称`回环接口`, 往往被分配打平 127.0.0.1 这个地址、经过内核处理后直接返回、不会在任何网络中出现.

#### MAC地址
`MAC地址`是一个网卡的物理地址、十六进制表示、6个byte、全局唯一. 但不可替代ip地址.
类似: 身份证号全网唯一、但若在问路时、被人知道的概率确很小、ip具有定位功能、类似省市区街道小区的概念、

MAC地址本身更像是身份证、是唯一标识. 设计唯一性是为了不同网卡放在同一网络时、可以不用担心冲突. 从硬件角度、确保不同网卡有不同标识.

MAC具有一定定位功能、但范围有限. 局限在同一子网. 跨子网的情况、就需要ip地址来查找了.

#### 网络设备的状态标识
<BROADCAST,MULTICAST,UP,LOWER_UP> 是干什么的？这个叫作 net_device flags，网络设备的状态标识

UP 代表网卡处于启动的状态
BROADCAST 表示网卡有广播地址、可以发送广播
MULTICAST 表示网卡可以发送多播包
LOWER_UP 表示L1是启动的, 即: 网线插着呢

mtu 1500 呢 ? 代表最大传输MTU为1500, 这个是以太网的默认值
网络是层层封装的, MTU是二层MAC概念. MAC层有MAC层的头、以太网规定连MAC头带正文合起来不能超过1500个字节、正文里有ip、tcp、http的头, 
若放不下、就需要分片来传输.

`qdisc pfifo_fast` 全称`queueing discipline`, 即`排队规则`. 内核若需要通过某个网络接口发送数据包、就需要按照为这个接口配置的qdisc将数据包加入队列.
最简单的qdisc是 pfifo, 它不对进入的数据包做任何处理、数据包采用先入先出的方式通过队列. pfifo_fast 稍微复杂些、它的队列包括3个波段(band)、在每个波段里使用先进先出的规则. band 0的优先级最高, band 2的最低, 若band 0里有数据包、系统就不会处理band 1里的数据包.

数据包是按照服务类型(`Type of Service, TOS`)被分配到三个波段的、TOS是 IP 头里边的一个字段、代表了当前包的优先级.
