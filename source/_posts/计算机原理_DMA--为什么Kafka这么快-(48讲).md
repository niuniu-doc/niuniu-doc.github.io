---
title: DMA--为什么Kafka这么快-(48讲)
date: 2020-03-20
categories:
  - 计算机原理
tags:
  - 计算机原理
---
> SSD的IOPS可以达到2w、4w, 可CPU主频在2GHZ以上、每秒可以有2亿次操作, 若对于IO操作、都是由CPU发出对应的指令、然后等待IO设备完成操作之后返回、那CPU有大部分的时间在等待IO. 其实对于IO设备的大量操作、都只是把内存数据传输到IO设备而已、CPU是无效等待, 于是, 就有了 `DMA`(Direct Memory Access)`直接内存访问` 技术

#### 理解DMA、一个协处理器
```
本质上: DMA技术就是在主板上放一块独立的芯片, 在进行内存与IO设备的数据传输时, 不在通过CPU来控制
数据传输、而是通过`DMA控制器`(`DMAC`), 这块芯片可以认为就是一个`协处理器`

DMAC最有价值的地方体现在, 当要传输的数据特别大、速度特别快, 或者传输的数据特别小、速度特别慢时.
eg. 1. 用千兆网卡或者硬盘传输大量数据时、若都用CPU来搬运的话、肯定忙不过来、所以, 选择DMAC
    2. 数据传输很慢时, DMAC可以等数据到齐了、再发送信号给CPU处理、而不是让CPU在那里忙等待.

DMAC在控制数据传输时、还是需要CPU的.

总线上的设备包括`主设备`和`从设备`, 主动发起数据传输的、必须是主设备, eg.CPU;从设备只能接收数据. 
所以: 只能是 CPU从IO设备读数据、或者是CPU向IO设备写数据.

IO设备不能向主设备发起请求吗？
可以. 但发送的不是实际的数据内容, 而是控制信号. IO设备可以告诉CPU, 有数据要传输、实际数据由CPU
拉取, 而不是IO设备主动推送给CPU.
```
![包含DMAC的数据传输.png](https://upload-images.jianshu.io/upload_images/14027542-c4145e8dae426f5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
DMAC既是一个主设备、有是一个从设备. 对CPU来说、它是从设备, 对IO设备来说、它是主设备.
使用DMAC进行数据传输的过程如下:
1. CPU作为一个主设备、向DMAC设备发起请求(就是在DMAC里修改配置寄存器).
2. CPU修改DMAC寄存器的时候, 会告诉DMAC几个信息:
   1) 源地址的初始值及传输时的地址增减方式
       源地址: 数据要从哪里传输过来. eg. 从内存写入硬盘时, 就是一个内存地址
              从硬盘读取到内存时, 就是硬盘的IO接口的地址.
       地址的增减方式: 是说数据从大的地址向小的地址传输, 还是从晓得地址往大的地址传输.
   2) 目标地址初始值及传输时的地址增减方式
   3) 要传输的数据长度
3. 设置完这些信息DMAC就变成一个空闲的状态idle
4. 若从硬盘 -> 内存加载数据, 硬盘会向DMAC发起一个数据传输请求,这个请求是通过额外的连线(不是总线)
5. DMAC再通过一个额外的连线响应该请求.
6. 于是, DMAC再向内存发起总线读的数据传输请求, 就从硬盘读到了DMAC控制器里.
7. 然后、DMAC再向我们的内存发起写的请求、把数据写入内存
8. DMAC会反复进行6、7 的操作, 直到DMAC的寄存器里边设置的数据长度传输完成.
9. 数据传输完成之后、DMAC重新回到第三步的空闲状态.

整个数据传输的过程中，我们不是通过 CPU 来搬运数据，而是由 DMAC 这个芯片来搬运数据.
但是 CPU 在这个过程中也是必不可少的. 因为传输什么数据，从哪里传输到哪里，其实还是 CPU 来设置的.
这也是为什么，DMAC 被叫作`协处理器`

早期的计算机里没有DMAC、所有数据都是CPU来搬运的、然后出现了主板上独立的DMAC控制器.
现在数据传输要求越来越复杂、加上显卡、网卡、硬盘等各个设备对数据传输的需求不同、各个设备
都有自己的DMAC芯片了
```

#### Kafka为什么这么快
```
kafka 是一个用来处理实时数据的管道, 常用来做消息队列、或者收集和落地海量的日志. 
作为一个实时数据和日志的管道、瓶颈自然也在IO层面

kafka常见的两种海量数据传输: 1.从网络接收上游数据,落地到本地磁盘 2.从本地磁盘读取、发送到网络上.

先看场景2, 最直观的是用一个文件读操作, 从磁盘把数据读到内存、再通过一个socket、把数据发到网络上.

File.read(fileDesc, buf, len);
Socket.send(socket, buf, len);

这样会有四次数据传输:(两次DMA、两次通过CPU控制的传输)
1. 从硬盘上、读到操作系统内核缓冲区(通过DMA搬运).
2. 将内核缓冲区的数据、复制到应用分配的内存里(通过CPU搬运)
3. 从应用的内存、写到操作系统Socket的缓冲区里(CPU搬运)
4. 从Socket的缓冲区、写到网卡的缓冲区(DMA)
```
![本次磁盘到网络的数据传输过程.png](https://upload-images.jianshu.io/upload_images/14027542-1827f1506ed4ab8f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
其实只需要搬运一份数据、却搬运了4次、且: 从内核的缓冲区传输到应用的内存里、再从应用的内存里
传输到socket的缓冲区里、其实是把一份数据在内存里搬来搬去, 所以kafka就调用Java NIO库、
FileCahnnel->transfer to 方法, 将数据直接通过Channel写入到对应网络设备, 且对于socket的操作、
也不是写到socket的buffer里、而是直接根据描述符写入到网卡的缓冲区, 省去了2、3, 只有两次数据传输.
```
![kafka的数据传输模型.png](https://upload-images.jianshu.io/upload_images/14027542-b190e4dc04bd2a64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
这样、同一份数据的传输次数从4次变成了2次、且没有通过CPU传输、都是DMA传输, 在这个方法里、
没有在内存层面copy数据, 也称为`零拷贝`(Zero-Copy)

```
