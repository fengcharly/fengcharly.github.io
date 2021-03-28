---
title: RPC服务(一) RPC相关概念详解，以及如何设计一个RPC框架
categories:
 - RPC
tags:
 - 后端开发
description: RPC是远程过程调用（Remote Procedure Call）的缩写形式。RPC的概念与技术早在1981年由Nelson提出...
---

#### 1.RPC是什么

​		RPC是远程过程调用（Remote Procedure Call）的缩写形式。RPC的概念与技术早在1981年由Nelson提出。
​		1984年，Birrell和Nelson把其用于支持异构型分布式系统间的通讯。Birrell的RPC 模型引入存根进程( stub) 作为远程的本地代理，调用RPC运行时库来传输网络中的调用。
​		Stub和RPC runtime屏蔽了网络调用所涉及的许多细节，特别是，参数的编码/译码及网络通讯是由stub和RPC runtime完成的，因此这一模式被各类RPC所采用。

​		简而言之，RPC就是“像调用本地方法一样调用远程方法”。

#### 2.RPC原理

RPC的简化版原理如下图所示，核心是代理机制。主要包含以下几个方面（注意异常处理）：

1. 本地代理存根: Stub
2.  本地序列化反序列化
3. 网络通信
4. 远程序列化反序列化
5. 远程服务存根: Skeleton
6. 调用实际业务服务
7. 原路返回服务结果
8. 返回给本地调用方

[![6z6UUO.png](https://z3.ax1x.com/2021/03/27/6z6UUO.png)](https://imgtu.com/i/6z6UUO)

#### 3.流行的RPC框架

很多语言都内置了RPC技术。
Java RMI
.NET Remoting
远古时期，就有很多尝试，

- Corba（Common ObjectRequest Broker Architecture）公共对象请求代理体系结
  构，OMG组织在1991年提出的公用对象请求代理程序结构的技术规范。底层结构是基于
  面向对象模型的，由OMG接口描述语言(OMG Interface Definition Language，OMG
  IDL)、对象请求代理(Objec tRequest Broker，ORB)和IIOP标准协议（Internet Inter
  ORB Protocol，也称网络ORB交换协议）3个关键模块组成。
- COM（Component Object Model，组件对象模型）是微软公司于1993年提出的一种
  组件技术，它是一种平台无关、语言中立、位置透明、支持网络的中间件技术。很多老
  一辈程序员心目中的神书《COM本质论》。

#### 4.如何设计一个RPC框架

[![6zcIYD.png](https://z3.ax1x.com/2021/03/27/6zcIYD.png)](https://imgtu.com/i/6zcIYD)



​		既然RPC的核心是代理，那我们在设计RPC框架的时候有必要考虑：使用JDK动态代理还是用AOP？RPC还用到了共享接口，序列化也是我们需要考量的：XML、文本、JSON、还是二进制？传输协议：基于TCP还是HTTP？结构如下所示：

  		1. 共享：POJO实体类定义，接口定义。
    		2. RPC是基于接口的远程服务调用。Java下，代理可以选择动态代理，或者AOP实现。
      		3. 序列化和反序列化的选择：
       - 语言原生的序列化，RMI，Remoting
       - 二进制平台无关，Hessian，avro，kyro，fst等
       - 文本，JSON、XML等
		4. 网络传输：
     -  TCP/SSL
     -  HTTP/HTTPS
		5. 通过接口查找具体的业务服务实现。











