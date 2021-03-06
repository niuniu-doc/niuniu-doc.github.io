---
title: 超线程和SIMD(27讲)
date: 2020-03-20
categories:
  - 计算机原理
tags:
  - 计算机原理
---
**回顾**

> `超标量`: 可以让取指令及指令译码并行进行、
>
> `VLIW`: 可以搞定指令先后依赖关系、使得一次取一个指令包

最后两个提升CPU性能的设计: `超线程` 和 `SIMD(单指令多数据流)`

#### 超线程

```
奔腾4处理器失败的重要原因: CPU的流水线级数太深、高达20级、而后期Prescott更是达到了31级、超长流水线、使得各种冒险、并发的方案都用不上

解决冒险、提升并发的方案, 本质上是一种 `指令级并行(Instruction-level parallelism)`
即: CPU 想要在同一个时间、并行的执行两条指令、而代码原本是有先后顺序的、流水线太深的话、之前的分支预测、乱序执行等优化方式都无法起到很好的效果、

2002年底、Pentium4的CPU、第一次引入了超线程(Hyper-Threading技术)
`超线程`: 既然CPU运行在代码层面有前后依赖关系的指令、会遇到各种冒险问题、如果运行完全独立的程序呢? 指令间是没有依赖关系的、那么跟多个CPU运行不同任务、或者多进程、多线程技术有什么区别呢 ?

1.多个CPU上运行不同的程序、单个CPU核心里切换运行不同的线程任务、在同一时间点上、其实一个物理的CPU核心只运行一个线程的指令、所以, 并没有真正做到指令级并行
2. 超线程 是把一个物理CPU核心、伪装成两个逻辑CPU、这个CPU会在硬件层面增加很多电路、使得我们可以在一个CPU核心内部、维护两个不同线程的指令状态信息
eg. 在一个物理CPU核心内部、有双份PC寄存器、指令寄存器及条件码寄存器、可以维护两条并行指令的状态、在外面看起来、好像有两个逻辑层面的CPU同时在运行、也叫`同时多线程`(Simultanneous Multi-Threading, 简称SMT)技术

但: 在CPU其它的功能组件上、eg. 指令译码器和ALU 依然只有一份、因为超线程并不真正运行两个指令、它的目的在于 一个线程A的指令、在流水线停顿的时候、让线程B与执行、利用此时CPU译码器和ALU的空闲来处理、提高CPU利用率

通过很小的CPU代价、实现了同时运行多个线程的效果、通常只要在CPU核心添加10%左右的逻辑功能、增加可以忽略不计的晶体管数量、就可以实现、但同样只能应用在特殊的额应用场景下(线程等待时间较长)
eg. 需要对很多请求的数据库应用、就很适合、各个指令都需要等待访问内存数据，CPU计算并未跑满、但当前指令往往要停顿在流水线上、等待内存数据返回、这时、让CPU里的各个功能单元、处理另一个db查询的案例就很好

```
![image.png](https://upload-images.jianshu.io/upload_images/14027542-3b63d476f142a99f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### SIMD: 如何加速矩阵算法

```
SIMD: 单指令多数据流(Single Instruction Multiple Data)
SISD: 单指令单数据(Single Instruction Single Data)
多核CPU可以实现MIMD(Multiple Instruction Multiple Data)

为什么SIMD指令可以提高代码执行效率呢? 
1.因为SIMD在获取数据和执行指令的时候都是并行的、从内存读数据的时候、SIMD是一次性读多个数据
Intel在引入SSE指令集的时候、在CPU里添加了8个128Bits的寄存器、即 16Bytes、一个寄存器一次性可以加载4个整数、比起循环加载、时间就节省了
2. 指令执行层面、4个整数加法、相互之间完全无依赖、也就没有冒险问题要处理、有足够多的FU即可并行执行、这个加法就是4路同时并行的、也会节省时间

基于SIMD的向量计算指令、正是在Intel发布Pentium处理器的时候、被引入的指令集、当时的指令集叫 `MMX`, 即: `Matrix Math eXtensions`的英文缩写、中文名字叫 矩阵数学扩展

```
![image.png](https://upload-images.jianshu.io/upload_images/14027542-f32a5c0c67a98d4f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



> 超线程技术是线程级并行的解决方案、很多场景下不一定能带来性能的提升
>
> SIMD是指令级并行的解决方案、在处理向量计算的情况下、同一个向量的不同维度之间的计算是相互独立的、
>
> 两者都优化了CPU的使用
