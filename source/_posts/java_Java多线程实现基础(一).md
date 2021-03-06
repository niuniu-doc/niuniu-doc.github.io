---
title: Java多线程实现基础(一)
date: 2020-03-20
categories:
  - Java
tags:
  - Java
---

#### CPU 的术语介绍
`内存屏障`: `memory barriers` 一组处理器指令、用于实现对内存操作的顺序限制

`缓冲行`: `cache line` cpu高速缓冲中可以分配的最小存储单位
         处理器填写缓冲行时、会加载整个缓存行、现代CPU需要执行几百次CPU指令

`原子操作`: `atomic operations` 不可中断的一个或者一系列操作

`缓冲行填充`: `cache line fill`当处理器识别到从内存中读取的操作数是可缓存的, 
        处理器读取整个高速缓存行到适当的缓存(L1,L2,L3或者所有)
        
`缓存命中`: `cache hit`如果进行高速缓存行填充操作的内存位置仍然是下次处理器访问的
地址时、处理器从缓存行读取而不是从内存读取

`写命中`: `write hit` 当处理器将操作数写回到内存缓存时、会先检查这个缓存的内存地址
是否在缓存行中、如果存在一个有效的缓存行、处理器会将这个操作数歇会到缓存、而不是内存

`写缺失`: `write misses the cache` 一个有效的缓存行被写入到不存在的内存区域

`比较并交换`: `Compare and Swap` cas操作需要两个值, old and new、在操作期间会比较这
两个值、如果发生变化则不交换、未发生变化才交换

`CPU流水线`: `CPU pipeline`在CPU中由5、6个不同的电路单元组成一条指令处理流水线
然后一条X86指令分成5~6步后再由这些电路单元分别执行、就能实现在一个CPU周期内完成
一条指令、提高CPU的运算速率

`内存顺序冲突`: `memory order violation`一般是由假共享引起的、是指多个CPU同时修改
同一个缓存行的不同部分而引起其中一个CPU的操作无效、当出现这个内存顺序冲突时、CPU必须清空流水线



#### Lock前缀的指令
```
lock 前缀的指令、在多核处理器下会引发两件事:
1. 将当期处理器的缓存行数据写回内存
2. 这个写回内存操作会使在其它CPU里缓存了该地址的数据无效
```

#### volatile的实现
```
为了提高速度、CPU不直接和内存通信、而是先将系统内存的数据加载到内存缓存后再操作
但: 操作完全不知道何时会被写回内存

volatile声明的变量进行读写、JVM会向处理器发送一条Lock前缀的指令、精讲变量所在的
缓存行数据写回到内存

在多核处理器下、实现了缓存一致性协议、每个处理器通过嗅探在总线上传播的数据来检查
自己的缓存是否已过期、当处理器发现自己缓存行对应的内存地址呗修改、会将当期处理器的
缓存行状态设为无效、重新从系统内存读取
```
