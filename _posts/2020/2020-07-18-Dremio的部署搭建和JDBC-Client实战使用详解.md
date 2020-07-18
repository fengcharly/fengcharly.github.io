---
title: Dremio的部署搭建和JDBC-Client实战使用详解
categories:
 - Java后端
tags:
 - 后端开发
description: Dremio的数据湖引擎提供了闪电般的查询速度和一个直接针对数据湖存储的自服务语义层...
---

#### 1.关于Dremio

Dremio的数据湖引擎提供了闪电般的查询速度和一个直接针对数据湖存储的自服务语义层。

- 闪电般的查询
- 自助服务语义层
- 灵活性和开源技术
- 强大的连接能力

更为详细的介绍请查阅官网文档:

```java
https://docs.dremio.com/
```

![UVeBAx.png](https://user-gold-cdn.xitu.io/2020/7/18/1735fe5c0a6f8c91?w=590&h=342&f=png&s=34991)

![UVZ2T0.png](https://user-gold-cdn.xitu.io/2020/7/18/1735fe5c0ad6a3d4?w=944&h=449&f=png&s=95429)

#### 2.在Linux上部署Dremio

##### ①获取Dremio安装包

```java
mkdir /data
cd    /data
wget http://download.dremio.com/community-server/3.1.8-201903290151120189-36bb2bf/dremio-community-3.1.8-201903290151120189_36bb2bf_1.noarch.rpm
```

下载比较慢的时候,可以尝试打开浏览器输入以上网址手动下载

##### ②使用rpm安装

```java
rpm -ivh dremio-community-3.1.8-201903290151120189_36bb2bf_1.noarch.rpm
```

##### ③启动Dremio

```java
sudo service dremio start
```

##### ④访问地址

```java
http://服务器IP:9047
```

注意: 首次启动可能需要注册,按照相应的要求填写即可,主要关注用户名和密码,随后见到如下登录页面		进行登录

 ![UVuyCV.png](https://user-gold-cdn.xitu.io/2020/7/18/1735fe5c0ceaf40b?w=963&h=504&f=png&s=27833)

##### ⑤设置数据源

数据源的设置Sources - > 添加(点击旁边的+号),可添加的数据源如下所示,按照指示配置即可

![UVKe5q.png](https://user-gold-cdn.xitu.io/2020/7/18/1735fe5c0d77f717?w=829&h=532&f=png&s=67707)

#### 3.使用SpringBoot+Mybatis连接使用Dremio

		由于暂时没有支持Dremio的连接池,我们这里使用的是Dremio的JDBC单连接,需要提前下载好Dremio的JDBC的jar包,地址如下:

```java
https://www.dremio.com/drivers/
```

	![UVlYIs.png](https://user-gold-cdn.xitu.io/2020/7/18/1735fe5c0cf97f14?w=1057&h=496&f=png&s=37918)
	
		这里我们是为了方便使用mybatis生成语句查询Dremio,当然如果你也可以选择Dremio的RestAPi来获取数据,相关介绍可以到如下网站查阅:

```java
https://docs.dremio.com/rest-api
```

##### 开始使用:

##### ①引入依赖

```java
<dependency>
 	<groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.0</version>
</dependency>
```

##### ②加入数据配置

```java
@Configuration
@MapperScan("com.demo.dao")
public class DataSourceConfig {
    @Bean
    public DataSource dataSource() {
        SimpleDriverDataSource dataSource = new SimpleDriverDataSource();
        dataSource.setDriverClass(com.dremio.jdbc.Driver.class);
        dataSource.setUsername("root");
        dataSource.setUrl("jdbc:dremio:direct=IP:31010;schema=test-data.TEST");
        dataSource.setPassword("*****");
        return dataSource;
    }

    @Bean
    public DataSourceTransactionManager transactionManager() {
        return new DataSourceTransactionManager(dataSource());
    }

    @Bean
    public SqlSessionFactoryBean sqlSessionFactory() throws Exception {
        SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(dataSource());
        sessionFactory.setTypeAliasesPackage("com.demo.model");
        return sessionFactory;
    }
}

```

##### ③编写DAO

```java
@Mapper
public interface DemoDao {


    String SQL_COLUMN_DEMO = "  ID, NAME, AGE ";

    @Select("<script>" +
            "select "+SQL_COLUMN_DEMO+"from demo where id = ${id}"
            + "</script>")
    Demo SelectOne(@Param("id")Long id);
}

```

#### 4.总结

以上省略了其余的逻辑相关,大家可以根据相关的业务进行完善.使用Dremio确实省去了很多和大数据联调的不便,它的快速查询特性也能提高查询效率,但是有利也有弊,使用时请注意以下相关问题:

- 暂无Dremio的连接池包,故用单连接
- 使用Dremio不支持预编译,请使用"${}"方式查询,注意手动控制SQL注入

- 只能用于查询,插入修改等操作暂不支持





 



