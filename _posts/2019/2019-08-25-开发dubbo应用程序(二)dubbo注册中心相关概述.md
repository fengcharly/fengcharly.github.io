---
title: 开发dubbo应用程序(二)dubbo注册中心相关概述
categories:
 - Dubbo
tags:
 - 后端开发
description: 在Dubbo微服务体系中,注册中心是其核心组件之一.Dubbo通过注册中心实现了分布式环境中各微服务之间的注册与发现,是各分布式节点之间的纽带....
---

### 1.注册中心概述

	在Dubbo微服务体系中,注册中心是其核心组件之一.Dubbo通过注册中心实现了分布式环境中各微服务之间的注册与发现,是各分布式节点之间的纽带.其主要作用如下:

- 动态加入。一个服务提供者通过注册中心可以动地把自己暴露给其他消费者,无序消费者逐个去更新配置文件;
- 动态发现。一个消费者可以动态地感知新的配置、路由规则和新的服务提供者，无需重启服务使之生效；
- 动态调整。注册中心支持参数的动态调整，新参数自动更新到所有相关服务节点；
- 统一配置。避免了本地配置导致每个服务的配置不一致问题。

Dubbo的注册中心源码在模块dubbo-register中，里面包含了五个子模块，如下所示：

| 模块名称                 | 模块介绍                          |
| ------------------------ | --------------------------------- |
| dubbo-register-api       | 包含了注册中心所有API和抽象实现类 |
| dubbo-register-zookeeper | 使用zookeeper作为注册中心的实现   |
| dubbo-register-redis     | 使用redis作为注册中心的实现       |
| dubbo-register-default   | Dubbo基于内存的默认实现           |
| dubbo-register-multicast | multicast模式的服务注册与发现     |

### 2.工作流程

- 服务启动时，会向注册中心写入自己的元数据信息，同时会订阅配置元数据；
- 消费者启动时也会像注册中心写入自己的元数据，并订阅服务提供者、路由和配置元数据信息；
- 服务治理中心（dubbo-admin），会同时订阅所有消费者、服务提供者、路由和配置元数据信息；
- 当所有服务提供者离开或有新的服务提供者加入是，注册中心服务提供者目录会发生变化，变化信息会动态通知给消费者、服务治理中心；
- 当消费方发起服务调用时，会异步将调用、统计信息等上报给监控中心（dubbo-monitor-simple）。

![mcIxZn.png](https://s2.ax1x.com/2019/08/25/mcIxZn.png)

### 3.Zookeeper原理概述

Zookeeper是树形结构的注册中心，每个节点的类型分为持久节点、持久顺序节点、临时节点和临时顺序节点。

- 持久节点：服务注册后保证节点不会丢失，注册中心重启也会存在；
- 持久顺序节点：在持久节点特性的基础上增加了节点先后顺序的能力；
- 临时节点：服务注册后连接丢失或session超时，注册的节点会自动被移除；
- 临时顺序节点：在临时节点特性的基础上增加了节点先后顺序的能力。

Dubbo使用Zookeeper作为注册中心时，只会创建持久节点和临时节点两种，对创建顺序并没有要求。

dubbo早zookeeper中的树形结构如下所示：

```java
+ /dubbo

+ -- servicce 

       +--providers

       +--consumers

      +--routers

   	+--configurators

```

### 4.树形结构关系

（1）树的根节点是注册中心分组，下面有多个服务接口，分组值来自用户配置\<dubbo:register>中的group属    性；

（2）服务接口下包含4类子目录，分别是providers、consumers、routes、configurators，这个路径是持久节点；

（3）服务提供者目录（/dubbo/service/providers）下面包含的接口有多个服务者URL元数据信息；

（4）服务消费者目录（/dubbo/service/consumers）下面包含的接口有多个消费者URL元数据信息；

（5）路由配置目录（/dubbo/service/routes）下面包含多个消费者路由策略URL元数据信息。

（6）动态配置目录（/dubbo/service/configurators）下面包含多个用于消费者由策略URL元数据信息。

在Dubbo中启用注册中心可参考如下方式：

```java
<beans>
<!--适用于Zookeeper一个集群有多个节点,多个IP和端口用逗号分隔-->
   <dubbo:registry protocol="zookeeper" address="ip:port,ip:port" />
 <!--适用于Zookeeper一个集群有多个节点,多个IP和端口用竖线分隔-->
  <dubbo:registry protocol="zookeeper" address="ip:port|ip:port" />
</beans>

```