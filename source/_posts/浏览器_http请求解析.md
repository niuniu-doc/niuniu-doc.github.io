---
title: http请求解析
date: 2020-03-20
categories:
  - 浏览器
tags:
  - 浏览器
---

#### 如何把数据包送达目的主机
> 主机A -> 网络层(添加IP头信息) -> 底层 -> 物理网络 -> 主机B -> 解析IP头信息 -> 将数据部分交给应用

#### 如何把数据包送达应用程序
> UDP协议: User Datagram Protocol
> 网络层和上层之间添加一层(传输层)、封装UDP头信息

UDP并不提供重传机制、只是丢弃当前数据包、且发送之后不知道是不是数据能到达目的地

#### TCP将数据完整送达应用程序
> 对于数据包丢失的情况、TCP提供重传机制
> 引入数据包排序机制、保证将乱序的数据包组合成一个完整的文件



```
浏览器使用HTTP协议作为应用层协议、用来封装请求的文本信息
使用TCP/IP作为传输层协议 将它发在网络上
即: HTTP请求的内容是通过TCP的传输数据阶段来实现的
```



#### HTTP请求流程

- 构建请求

> GET /index.html HTTP1.1

- 查找缓存

> 浏览器在网络请求之后会保存资源副本在本地
>
> 1. 缓解服务端压力、提升性能
> 2. 对于网站来说、可以实现快速下载
> 3. 若本地无资源副本、就会进入网络请求

- 准备IP地址和端口

> 思考：
>
> 1. http请求的第一步是什么呢 ？- 构建请求信息
> 2. 建立连接的信息都包含什么? - ip和端口号
> 3. 如果只有url可以拿到建立连接的信息么 ？- 通过DNS解析、http协议默认80端口

- 等待TCP队列

> 准备好IP和端口、是不是就可以建立TCP连接？
>
> 不一定, Chrome 同一个域名最多建立6个TCP连接、若同时有10个请求、会有4个请求进入排队队列等待、直到进行中的请求完成

- 建立TCP连接

> 在http开始工作之前、需要先建立TCP连接

- 发送HTTP请求

> 1. 请求行(请求方法、路由、http版本)
> 2. 请求头(cookie等信息)
> 3. 请求体(请求参数等)



#### 服务端处理http请求流程

* 返回请求
* 断开连接(正常情况下、一单server返回响应数据、就会关闭TCP连接、若头信息加入 Connection:Keep-Alive则保持TCP连接不断开)
* 重定向
