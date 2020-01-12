---
title: 分布式任务调度平台XXL-JOB快速使用与问题总结
categories:
 - Java后端
tags:
 - 后端开发
description: XXL-JOB是一个轻量级分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用...
---

#### 1.XXL-JOB简介

> XXL-JOB is a lightweight distributed task scheduling framework. It's core design goal is to develop quickly and learn simple, lightweight, and easy to expand. Now, it's already open source, and many companies use it in production environments, real "out-of-the-box".
>
> XXL-JOB是一个轻量级分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。
>
> 																							——引用自XXL-JOB的GIT项目介绍

![QEOvzq.png](https://user-gold-cdn.xitu.io/2019/11/30/16ebb157e4766446?w=1905&h=778&f=png&s=94439)

		关于XXL-JOB的其他介绍可以[参考官网中文文档介绍](https://www.xuxueli.com/xxl-job/)，它含有丰富的特性，支持分布式和动态任务添加以及权限控制等。

#### 2.搭建XXL-JOB项目

##### ①下载源码

```java
https://github.com/xuxueli/xxl-job/
```

##### ②执行SQL

```java
xxl-job-2.1.1\xxl-job-2.1.1\doc\db\tables_xxl_job.sql
```

##### ③修改配置

```java
### xxl-job, datasource
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/xxl_job?Unicode=true&characterEncoding=UTF-8
spring.datasource.username=root
spring.datasource.password=root_pwd
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
### xxl-job, access token 秘钥填了 下面的子项目要和这个一致
xxl.job.accessToken=
```

需要注意的是子项目中的配置地址要和admin中的访问首页地址一致:

```java
### xxl-job admin address list, such as "http://address" or "http://address01,http://address02"
xxl.job.admin.addresses=http://127.0.0.1:8080/xxl-job-admin
```

##### ④启动项目

		我们首选需要启动项目中的admin,然后再启动xxl-job-executor-samples下面的内容,这里我们启动,这里比较简单的方式是通过springboot的例子进行启动,这也是作者推荐的启动方式。
	
	  启动后访问调度中心访问地址：http://localhost:8080/xxl-job-admin (该地址执行器将会使用到，作为回调地址)

默认登录账号 “admin/123456”, 登录后运行界面如下图所示

![QV3nw6.png](https://user-gold-cdn.xitu.io/2019/11/30/16ebb157da46ac64?w=1908&h=922&f=png&s=85804)

关于job的配置可以参考Demo示例，然后我们添加时候用BEAN模式，名称为@Service的名称：

```java
/**
 * 任务Handler示例（Bean模式）
 *
 * 开发步骤：
 * 1、继承"IJobHandler"：“com.xxl.job.core.handler.IJobHandler”；
 * 2、注册到Spring容器：添加“@Component”注解，被Spring容器扫描为Bean实例；
 * 3、注册到执行器工厂：添加“@JobHandler(value="自定义jobhandler名称")”注解，注解value值对应的是调度中心新建任务的JobHandler属性的值。
 * 4、执行日志：需要通过 "XxlJobLogger.log" 打印执行日志；
 *
 * @author xuxueli 2015-12-19 19:43:36
 */
@JobHandler(value="demoJobHandler")
@Component
public class DemoJobHandler extends IJobHandler {

	@Override
	public ReturnT<String> execute(String param) throws Exception {
		XxlJobLogger.log("XXL-JOB, Hello World.");
		System.out.println("XXL-JOB, Hello World.");
		for (int i = 0; i < 5; i++) {
			XxlJobLogger.log("beat at:" + i);
			TimeUnit.SECONDS.sleep(2);
		}
		return SUCCESS;
	}
}
```

#### 3.问题总结

- **连接不上数据库?**<br/>需要在admin中配置Datasource相关连接,有密码需要填写正确。
- **执行日志一直处于执行中,回调失败?**<br/>配置子项目的时候一定要和admin的访问地址一致,用于回调。
- **找不到代码编写界面?**<br/>GLUE模式只在新增的时候选定了，才能使用界面编写代码，支持版本回退。
- **执行器管理不生效?**<br/>AppName要和配置里面的一致，手动添加的地址要和netty对外的端口号一致（不是tomcat秋启动端口号），默认为9999。

   

  

