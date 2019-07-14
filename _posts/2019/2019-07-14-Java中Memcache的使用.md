---
title: Java中Memcache的使用
categories:
 - Memcache
tags:
 - 后端开发
description: MyBatis-Plus（简称 MP）是一个 MyBatis 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生...
---

## 一、什么是Memcached？

        Memcached是danga.com开发的分布式内存对象缓存系统，所谓分布式，意味着它不是本地的，而是基于网络连接完成服务。Memcached把一些数据通过key=value数据存储到内存中，这样访问更加方便快捷。但是随之而来的问题是如果Memcached关闭或者Memcached的服务器关闭那么所保存的内容也就没有了。

## 二、安装Memcached服务端

使用以下地址下载:

```java
　http://downloads.northscale.com/memcached-win32-1.4.4-14.zip

　http://downloads.northscale.com/memcached-win64-1.4.4-14.zip
```

然后解压,在相应的文件夹下执行以下的命令启动memCache:

```java
memcached.exe -d install

memcached.exe -d start

```

使用memcached -h命令查看是否安装成功,出现以下的界面说明安装成功:

![ZIoSts.png](https://s2.ax1x.com/2019/07/14/ZIoSts.png)

## 三、java下使用Memcached（java客户端程序）

maven的依赖如下:

```java
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.demo</groupId>
  <artifactId>sso-practice</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>sso-practice Maven Webapp</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>

    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>1.7.12</version>
    </dependency>
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-core</artifactId>
      <version>1.1.1</version>
      <scope>provided</scope>
    </dependency>
    <!--实现slf4j接口并整合-->
    <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>1.1.1</version>
      <scope>provided</scope>
    </dependency>

    <!--druid==>阿里巴巴数据库连接池-->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.0.25</version>
    </dependency>
    <!--2.dao框架:MyBatis依赖-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.3.0</version>
    </dependency>
    <!--mybatis自身实现的spring整合依赖-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.2</version>
    </dependency>

    <!--3.Servlet web相关依赖-->
    <dependency>
      <groupId>taglibs</groupId>
      <artifactId>standard</artifactId>
      <version>1.1.2</version>
    </dependency>
    <dependency>
      <groupId>jstl</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>

    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.5.4</version>
    </dependency>

    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>

    <!--4:spring依赖-->
    <!--1)spring核心依赖-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>4.3.14.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>4.3.14.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>4.3.14.RELEASE</version>
    </dependency>
    <!--2)spring dao层依赖-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>4.3.14.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>4.3.14.RELEASE</version>
    </dependency>
    <!--3)springweb相关依赖-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>4.3.14.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>4.3.14.RELEASE</version>
    </dependency>
    <!--4)spring test相关依赖-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>4.3.14.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.35</version>
      <scope>runtime</scope>
    </dependency>

    <!--Aop要导入的包-->
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.8.9</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.json/json -->
    <dependency>
      <groupId>org.json</groupId>
      <artifactId>json</artifactId>
      <version>20170516</version>
    </dependency>

    <!-- slf4j -->
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-log4j12</artifactId>
      <version>1.7.5</version>
    </dependency>
      <!-- https://mvnrepository.com/artifact/de.flapdoodle.embed/de.flapdoodle.embed.memcached -->
      <dependency>
          <groupId>de.flapdoodle.embed</groupId>
          <artifactId>de.flapdoodle.embed.memcached</artifactId>
          <version>1.06.4</version>
          <scope>test</scope>
      </dependency>
    <dependency>
      <groupId>net.spy</groupId>
      <artifactId>spymemcached</artifactId>
      <version>RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>net.spy</groupId>
      <artifactId>spymemcached</artifactId>
      <version>RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>net.spy</groupId>
      <artifactId>spymemcached</artifactId>
      <version>RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>net.spy</groupId>
      <artifactId>spymemcached</artifactId>
      <version>RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>net.spy</groupId>
      <artifactId>spymemcached</artifactId>
      <version>RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>net.spy</groupId>
      <artifactId>spymemcached</artifactId>
      <version>RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>net.spy</groupId>
      <artifactId>spymemcached</artifactId>
      <version>RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>net.spy</groupId>
      <artifactId>spymemcached</artifactId>
      <version>RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>net.spy</groupId>
      <artifactId>spymemcached</artifactId>
      <version>RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>net.spy</groupId>
      <artifactId>spymemcached</artifactId>
      <version>RELEASE</version>
      <scope>compile</scope>
    </dependency>

    <!-- https://mvnrepository.com/artifact/commons-pool/commons-pool -->
    <dependency>
      <groupId>commons-pool</groupId>
      <artifactId>commons-pool</artifactId>
      <version>1.5.6</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/spy/memcached -->
    <dependency>
      <groupId>spy</groupId>
      <artifactId>memcached</artifactId>
      <version>2.4rc1</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/com.whalin/Memcached-Java-Client -->
    <dependency>
      <groupId>com.whalin</groupId>
      <artifactId>Memcached-Java-Client</artifactId>
      <version>3.0.2</version>
    </dependency>


    <!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-api -->
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>1.6.1</version>
    </dependency>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-api</artifactId>
      <version>RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.testng</groupId>
      <artifactId>testng</artifactId>
      <version>RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>net.spy</groupId>
      <artifactId>spymemcached</artifactId>
      <version>2.10.0</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
    </dependency>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-api</artifactId>
      <version>RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-api</artifactId>
      <version>RELEASE</version>
      <scope>compile</scope>
    </dependency>

  </dependencies>

  <build>
    <finalName>sso-practice</finalName>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
      <plugins>
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.1.0</version>
        </plugin>
        <!-- see http://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_war_packaging -->
        <plugin>
          <artifactId>maven-resources-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.8.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.22.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-war-plugin</artifactId>
          <version>3.2.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-install-plugin</artifactId>
          <version>2.5.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-deploy-plugin</artifactId>
          <version>2.8.2</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
</project>


```

### MemCacheTest:

```java
package com.demo.other;

import com.whalin.MemCached.MemCachedClient;
import com.whalin.MemCached.SockIOPool;
import org.junit.Test;

import java.util.Date;

public class MemCacheTest {

    @Test
    public  void show(){
        MemCachedClient client=new MemCachedClient();
        //使用的服务器，由于是在本地测试，只有一个服务器地址。默认端口是11211
        //格式为 服务器IP:端口号
        String [] addr={"127.0.0.1:11211"};
        /**
         * 设置权重，与设定的服务器一一对应
         */
        Integer[] weight={3};

        //建立通信的连接池
        SockIOPool pool=SockIOPool.getInstance();
        //设置连接池可用cache服务器列表，服务器构成形式：ip地址+端口号
        pool.setServers(addr);
        //设置连接池可用cache服务器的权重，和server数组的位置一一对应
        pool.setWeights(weight);


        //设置初始连接数
        pool.setInitConn(5);
        //设置最小连接数
        pool.setMinConn(5);
        //设置最大连接数
        pool.setMaxConn(200);
        //设置可用连接的最长等待时间
        pool.setMaxIdle(1000*30*30);
        //设置连接池维护线程的睡眠时间，设置为0，维护线程不启动
        pool.setMaintSleep(30);
        //设置Nagle算法，设置为false，因为通讯数据量比较大要求相应及时
        pool.setNagle(false);
        //设置socket读取等待超时时间
        pool.setSocketTO(30);
        //设置连接等待超时值
        pool.setSocketConnectTO(0);
        //设置完参数后，启动pool
        pool.initialize();

        client.set("value","Ok");
        String value= (String) client.get("value");

        //设置定时时间2s后消失
        client.set("value1","OK2",new Date(2000));
        String value2= (String) client.get("value1");
        System.out.println(value);
        System.out.println(value2);
    }
}


```



### 2、spring整合memcached     

```xml
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>classpath:property/memcached.properties</value>
            </list>
        </property>
    </bean>
    <!--spring整合memched-->
    <bean id="memcachedPool" class="com.danga.MemCached.SockIOPool" factory-method="getInstance"
          init-method="initialize"   destroy-method="shutDown">
        <property name="servers">
            <list>
                <value>${memcached.server}</value>
            </list>
        </property>

        <property name="maxConn" value="${memcached.maxConn}"></property>
        <property name="maintSleep" value="${memcached.maniSleep}"></property>
        <property name="nagle" value="${memcached.nagle}"></property>
        <property name="socketTO" value="${memcached.socketTO}"></property>
    </bean>
```

 properties配置文件：

```properties
#服务器地址
memcached.server=127.0.0.1:11211
#初始连接数目
memcached.initConn=20
#每个服务器建立最大连接数
memcached.maxConn=50
#自查线程周期工作，其每次休眠时间
memcached.maniSleep=3000
#是否使用nagle算法（Socket参数，如果是true，写数据不缓冲，直接发送）
memcached.nagle=false
#Socket阻塞读取数据的超时时间
memcached.socketTO=3000
```

### 测试：

```java
@RunWith(SpringJUnit4ClassRunner.class)//表示整合JUnit进行测试
@ContextConfiguration(locations ={"classpath:applicationContext.xml"})
public class SpringTest {

    @Test
    public void test1(){
        MemCachedClient memCachedClient=new MemCachedClient();
        memCachedClient.set("username","luck");
        String value= (String) memCachedClient.get("username");
        System.out.println(value);
    }
}
```