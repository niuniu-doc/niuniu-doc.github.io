---
title: Java多线程实现基础(二)
date: 2020-03-20
categories:
  - Java
tags:
  - Java
---
#### Java内存模型

#### 基本概念
```
1. 线程通信
   线程之间以何种方式进行消息传递
   共享内存 & 消息传递
   
2. 线程同步
   程序间用于控制不同线程间操作发生相对顺序的机制
   
   java是共享内存模型】采用的是隐式通信、显式修改
   
   jvm定义了线程和主内存(Main Memory)之间的关系:
   1) 线程之间的共享变量存储在主内存中
   2) 线程私有变量存储在私有本地内存 Local Memory中、
      本地内存是jvm的一个抽象概念、并非真实存在、涵盖缓存、缓冲区、寄存器及其它硬件及编译器优化
   Java同步原语(synchronize volatile  final)
```

### 指令重排
```
1. 编译器重排
   在不改变单线程语义的条件下、编译器可以重排指令的执行顺序
2. 指令重排 
   现代cpu采用了指令级并行技术、可以同时执行多条指令、若无数据依赖、
   处理器可以改变机器指令的执行顺序
3. 内存系统的重排
   由于处理器采用缓存和读写缓冲区、使得加载和存储操作看上去是乱序的、
   可能在乱序执行
```

#### jvm 指令执行
```
未使用同步的程序在jvm中的执行基本无序、
1. jvm不保证单线程内的操作会按照程序代码顺序执行、临界区指令重排
2. jvm不保证所有线程看到的执行顺序一致
3. jvm不保证64位的long、double类型写操作具有原子性

```
