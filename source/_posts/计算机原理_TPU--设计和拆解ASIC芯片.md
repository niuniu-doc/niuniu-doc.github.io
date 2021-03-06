---
title: TPU--设计和拆解ASIC芯片
date: 2020-03-20
categories:
  - 计算机原理
tags:
  - 计算机原理
---
> GPU天生适合海量、并行的矩阵运算、于是大量用在深度学习的模型训练上
深度学习中计算量最大的是什么呢 ? 深度学习的推断部分

```
`推断部分`: 在完成深度学习训练之后、把训练完成的模型存储下来. 这个存储下来的模型、是许多个向量组成的参数、然后根据这些参数、计算输入的数据、得到结果. eg. 推测用户是否点击广告; 扫身份证进行人脸识别
```

思考: 模型的训练和推断有什么区别 ?
```
一、深度学习的推断、灵活性要求更低. 只需要计算一些矩阵的乘法、加法、调用一些sigmoid这样的激活函数、可能计算很多层、但也只是这些计算的简单组合

二、深度学习推断的性能、首先要保证响应时间的指标
模型训练的时候、只需要考虑吞吐率就可以、但推断不行. eg. 我们不希望人脸识别会超过几秒钟

三、深度学习的推断工作、希望功耗尽可能的小一些
因为深度学习的推断要7*24小时的跑在数据中心、且对应芯片要大规模的部署在数据中心、一块芯片减少5%的功耗、就可以节省大量的电力

```
> 于是: 第一代TPU的设计目标:
在保障响应时间的情况下、尽可能的提高`能效比`这个指标、也就是进行相同数量的推断工作、花费的整体能源要低于CPU和GPU

#### TPU的几点设计
> 1. 向前兼容  2. TPU未设计成包含取指电路的GPU、而是通过CPU发送需要执行的指令
3. 使用`SRAM` 作为统一缓冲区, `SRAM`一般用来作为CPU的寄存器或者高速缓存、`SRAM`比`DRAM`快, 但因为电路密度小、占用空间大、价格也较贵、之所以选择`SRAM`是因为整个推断过程、它会高频反复地被矩阵乘法单元读写、来完成计算
4. 细节优化, 使用8Bits数据

![image.png](https://upload-images.jianshu.io/upload_images/14027542-9db6fb03181e5443.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
