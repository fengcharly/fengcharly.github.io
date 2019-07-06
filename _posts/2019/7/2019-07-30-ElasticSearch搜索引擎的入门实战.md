---
title: ElasticSearch搜索引擎的入门实战
categories:
 - ElasticSearch
tags:
 - 后端开发
description: ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎...
---

# ElasticSearch搜索引擎的入门实战

#### 1.ElasticSearch简介

> 引用自百度百科:
>
> ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。
>
> 我们建立一个网站或应用程序，并要添加搜索功能，但是想要完成搜索工作的创建是非常困难的。我们希望搜索解决方案要运行速度快，我们希望能有一个零配置和一个完全免费的搜索模式，我们希望能够简单地使用JSON通过HTTP来索引数据，我们希望我们的搜索服务器始终可用，我们希望能够从一台开始并扩展到数百台，我们要实时搜索，我们要简单的多租户，我们希望建立一个云的解决方案。因此我们利用Elasticsearch来解决所有这些问题及可能出现的更多其它问题。

#### 2.ElasticSearch的安装

##### 2.1.安装Jdk1.8

```shell
1、查看Linux环境 是否含有JDK 卸载 如果检查到有安装就执行卸载命令
  rpm -qa|grep jdk
  卸载命令
  rpm -e --nodeps 软件名称  

2、下载jdk
打开网站http://www.oracle.com/technetwork/java/javase/downloads，选择对应版本JDK，点击下载

3、本人下载jdk为jdk-8u131-linux-x64.tar.gz 上传并解压:
 tar -xvf jdk-8u131-linux-x64.tar.gz -C /usr/local/

4、进入解压缩目录

  cd /usr/local

5、修改jdk的文件夹名称

  mv jdk1.8.0_131  jdk

6、配置环境变量

   修改环境变量配置文件：  vi /etc/profile


   点 i键进入编辑模式

   跳转到最后一行
   增加如下内容

   #java runtime seting
export JAVA_HOME=/usr/local/jdk
export CLASSPATH=$JAVA_HOME/lib:.
export PATH=$JAVA_HOME/bin:$PATH


  按ESC 输入:wq  保存退出

7、重新加载环境配置

   source /etc/profile

 
8、测试JDK安装是否ok

  java -version
```

##### 2.2.安装elasticsearch-2.4.1

1、下载

```shell
wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.4.1/elasticsearch-2.4.1.tar.gz
```

2、解压 注意解压路径不能为root

```shell
tar -zxvf elasticsearch-2.4.1.tar.gz -C /usr/local
```

3、配置
找到解压后的config文件夹打开elasticsearch.yml

关键是要打开network.host: 0.0.0.0和http.port: 9200注释，提供web访问

```java
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please see the documentation for further information on configuration options:
# <http://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration.html>
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
# 集群名称，默认为elasticsearch
cluster.name: my-application
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
# 节点名称，es启动时会自动创建节点名称，但你也可进行配置
node.name: node-1
#
# Add custom attributes to the node:
#
# 是否作为主节点，每个节点都可以被配置成为主节点，默认值为true：
# node.master: true
#
# node.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
# 分配给当前节点的索引数据所在的位置
# path.data: /path/to/data
#
# Path to log files:
# 日志文件所在位置
# path.logs: /path/to/logs
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
# bootstrap.memory_lock: true
#
# Make sure that the `ES_HEAP_SIZE` environment variable is set to about half the memory
# available on the system and that the owner of the process is allowed to use this limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
network.host: 0.0.0.0
#
# Set a custom port for HTTP:
#
http.port: 9200
#
# For more information, see the documentation at:
# <http://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html>
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when new node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
# discovery.zen.ping.unicast.hosts: ["host1", "host2"]
#
# Prevent the "split brain" by configuring the majority of nodes (total number of nodes / 2 + 1):
#
# discovery.zen.minimum_master_nodes: 3
#
# For more information, see the documentation at:
# <http://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery.html>
#
# ---------------------------------- Gateway -----------------------------------
#
# Block initial recovery after a full cluster restart until N nodes are started:
#
# gateway.recover_after_nodes: 3
#
# For more information, see the documentation at:
# <http://www.elastic.co/guide/en/elasticsearch/reference/current/modules-gateway.html>
#
# ---------------------------------- Various -----------------------------------
#
# Disable starting multiple nodes on a single system:
#
# node.max_local_storage_nodes: 1
#
# Require explicit names when deleting indices:
#
# action.destructive_requires_name: true

```

4、设置文件可访问权限

```shell
chmod -R 777 elasticsearch-2.4.1
```

5、注意这里不能直接启动 用root用户启动会报错 所以我们需要新建一个用户

```shell
useradd admin
su admin
```

6、启动 在bin目录下执行命令(最后可加& 表示在后台运行)

```java
 ./elasticsearch -d 
```

7、查看内存

执行命令

```
 ps -ef|grep elasticsearch
```

![ZSI2Je.png](https://s2.ax1x.com/2019/06/21/ZSI2Je.png)

8、查看内存 优化内存

可以看到在内存为2G的主机上，Elasticsearch的运行内存为 `-Xms256m -Xmx1g`

1.最简单的一个方法就是指定ES_HEAP_SIZE环境变量。服务进程在启动时候会读取这个变量，并相应的设置堆的大小。设置命令如下：

```
export ES_HEAP_SIZE=1g
```

9、访问网页

因为开启了http访问，并设置可以远程连接，所以我们直接请求网址即可：

```
http://你的服务器IP:9200
```

相应内容如下:

```json
{
name: "Man-Spider",
cluster_name: "elasticsearch",
cluster_uuid: "hsuDHGiMQWaML6_xSJN8fw",
version: {
number: "2.4.1",
build_hash: "c67dc32e24162035d18d6fe1e952c4cbcbe79d16",
build_timestamp: "2016-09-27T18:57:55Z",
build_snapshot: false,
lucene_version: "5.5.2"
},
tagline: "You Know, for Search"
}
```

10、可视化插件

利用在bin目录下提供的plugin插件安装head，实现web可视化

注意目录：

```java
./bin/plugin install mobz/elasticsearch-head
```

安装完后访问

```
http://你的服务器IP:9200/_plugin/head/
```

![ZSodk8.png](https://s2.ax1x.com/2019/06/21/ZSodk8.png)

#### 3.使用logstash导入mysql数据到elasticSearch

Elasticsearch-jdbc工具包（废弃）,虽然是官方推荐的，但是已经几年不更新了。所以选择安装logstash-input-jdbc，首选 logstash-input-jdbc,logstash5.X开始，已经至少集成了logstash-input-jdbc插件。所以，你如果使用的是logstash5.X，可以不必再安装，可以直接跳过这一步。

##### 3.1.下载mysql-jdbc-driver.jar

下载地址:

```java
 jdbc连接mysql驱动的文件目录，可去官网下载:https://dev.mysql.com/downloads/connector/j/
```

此处我使用的jar的版本为:mysql-connector-java-5.1.46.jar.后面我们会把这个jar包放在logstash的config目录下面

##### 3.2.下载logstash-6.3.0 

下载命令:

```shell
sudo wget  https://artifacts.elastic.co/downloads/logstash/logstash-6.3.0.zip
```

解压命令:

```shell
yum install -y unzip
unzip logstash-6.3.0.zip
```

##### 3.3.elasticSearch与数据库表的对应关系

| ES   | MYSQL        |
| ---- | ------------ |
| 索引 | 数据库       |
| 类型 | 数据表       |
| 文档 | 数据表的一行 |
| 属性 | 数据表的一列 |

##### 3.4.建立测试数据表

sql语句如下:

```sql
DROP TABLE IF EXISTS `student`;
CREATE TABLE `student` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `update_time` datetime DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Records of student
-- ----------------------------
INSERT INTO `student` VALUES ('1', '调度', '12', '2019-06-13 19:24:55');
INSERT INTO `student` VALUES ('2', '李四', '13', '2019-06-13 19:24:55');
INSERT INTO `student` VALUES ('3', '王五', '15', '2019-06-13 19:24:55');
INSERT INTO `student` VALUES ('4', '赵六', '18', '2019-06-13 21:01:32');
INSERT INTO `student` VALUES ('5', '的地方', '52', '2019-06-13 21:13:51');
INSERT INTO `student` VALUES ('6', '测试', '45', '2019-06-13 21:17:31');

```

在logstatsh的目录下面建立my_logstash文件夹,里面建立myjdbc.conf:(这个仅供参考 实际不使用)

```java
input {
  jdbc {
    # mysql相关jdbc配置
    jdbc_connection_string => "jdbc:mysql://IP:3306/test?useUnicode=true&characterEncoding=utf-8&useSSL=false"
    jdbc_user => "****"
    jdbc_password => "****"

    # jdbc连接mysql驱动的文件目录，可去官网下载:https://dev.mysql.com/downloads/connector/j/
    jdbc_driver_library => "./config/mysql-connector-java-5.1.46.jar"
    # the name of the driver class for mysql
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_paging_enabled => true
    jdbc_page_size => "50000"

    jdbc_default_timezone =>"Asia/Shanghai"

    # mysql文件, 也可以直接写SQL语句在此处，如下：
    #statement => "select * from student where update_time >= :sql_last_value"
     statement => "select * from student"
    # statement_filepath => "./config/jdbc.sql"

    # 这里类似crontab,可以定制定时操作，比如每分钟执行一次同步(分 时 天 月 年)
    schedule => "* * * * *"
    #type => "jdbc"

    # 是否记录上次执行结果, 如果为真,将会把上次执行到的 tracking_column 字段的值记录下来,保存到      last_run_metadata_path 指定的文件中
    #record_last_run => true

    # 是否需要记录某个column 的值,如果record_last_run为真,可以自定义我们需要 track 的 column 名称，此时该参数就要为 true. 否则默认 track 的是 timestamp 的值.
    use_column_value => true

    # 如果 use_column_value 为真,需配置此参数. track 的数据库 column 名,该 column 必须是递增的. 一般是mysql主键
    tracking_column => "update_time"

    tracking_column_type => "timestamp"

    last_run_metadata_path => "./logstash_capital_bill_last_id"

    # 是否清除 last_run_metadata_path 的记录,如果为真那么每次都相当于从头开始查询所有的数据库记录
    clean_run => false

    #是否将 字段(column) 名称转小写
    lowercase_column_names => false
  }
}

output {
  elasticsearch {
    hosts => "192.168.142.128:9200"
    index => "resource"
    document_id => "%{id}"
    template_overwrite => true
  }

  # 这里输出调试，正式运行时可以注释掉
  stdout {
      codec => json_lines
  } 
}

```

在logstash的config目录下面新建logstash-mysql.conf:

```shell
input {
  jdbc {
    jdbc_driver_library => "/usr/local/logstash-6.3.0/config/mysql-connector-java-5.1.46.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://192.168.142.128:3306/test?characterEncoding=UTF-8&useSSL=false"
    jdbc_user => "root"
    jdbc_password => "root"
    statement => "SELECT * FROM student WHERE update_time > :sql_last_value"
    jdbc_paging_enabled => "true"
    jdbc_page_size => "50000"
    schedule => "* * * * *"
  }
}

filter {
   json {
        source => "message"
        remove_field => ["message"]
    }
}

output {
  stdout {
    codec => rubydebug
  }
  elasticsearch {
    hosts => "192.168.142.128:9200"   
    index => "test"
  }        
}

```

启动命令:

```java
 /usr/local/logstash-6.3.0/bin/logstash -f /usr/local/logstash-6.3.0/config/logstash-mysql.conf
```

启动成功后界面如下,开始导入数据:

![Zu0wTI.png](https://s2.ax1x.com/2019/06/27/Zu0wTI.png)

访问地址:<http://192.168.142.131:9200/_plugin/head/> 

![ZuDmZ9.png](https://s2.ax1x.com/2019/06/27/ZuDmZ9.png)

![ZuDMPx.png](https://s2.ax1x.com/2019/06/27/ZuDMPx.png)

#### 4.查询语句相关

##### URL查询:

```java
CURL GET http://192.168.142.131:9200/索引(test)/_search?q = name:李
```

##### 响应:

```java
{
    took: 5,
    timed_out: false,
    _shards: {
    total: 5,
    successful: 5,
    failed: 0
    },
    hits: {
    total: 6,
    max_score: 1,
    hits: [
            {
    _index: "test",
    _type: "doc",
    _id: "AWueHx85GWxHTP0ts5i7",
    _score: 1,
    _source: {
    name: "李四",
    id: 2,
    update_time: "2019-06-13T10:24:55.000Z",
    @version: "1",
    age: 13,
    @timestamp: "2019-06-28T12:46:04.976Z"
                }
            },
            {
    _index: "test",
    _type: "doc",
    _id: "AWueHx85GWxHTP0ts5i9",
    _score: 1,
    _source: {
    name: "赵六",
    id: 4,
    update_time: "2019-06-13T12:01:32.000Z",
    @version: "1",
    age: 18,
    @timestamp: "2019-06-28T12:46:04.977Z"
                }
            },
            {
    _index: "test",
    _type: "doc",
    _id: "AWueHx85GWxHTP0ts5i8",
    _score: 1,
    _source: {
    name: "王五",
    id: 3,
    update_time: "2019-06-13T10:24:55.000Z",
    @version: "1",
    age: 15,
    @timestamp: "2019-06-28T12:46:04.977Z"
                }
            },
            {
    _index: "test",
    _type: "doc",
    _id: "AWueHx85GWxHTP0ts5i_",
    _score: 1,
    _source: {
    name: "测试",
    id: 6,
    update_time: "2019-06-13T12:17:31.000Z",
    @version: "1",
    age: 45,
    @timestamp: "2019-06-28T12:46:04.978Z"
                }
            },
            {
    _index: "test",
    _type: "doc",
    _id: "AWueHx85GWxHTP0ts5i6",
    _score: 1,
    _source: {
    name: "调度",
    id: 1,
    update_time: "2019-06-13T10:24:55.000Z",
    @version: "1",
    age: 12,
    @timestamp: "2019-06-28T12:46:04.962Z"
                }
            },
            {
    _index: "test",
    _type: "doc",
    _id: "AWueHx85GWxHTP0ts5i-",
    _score: 1,
    _source: {
    name: "的地方",
    id: 5,
    update_time: "2019-06-13T12:13:51.000Z",
    @version: "1",
    age: 52,
    @timestamp: "2019-06-28T12:46:04.978Z"
                }
            }
        ]
    }
}
```

##### 相关参数

如下这些参数可以使用统一资源标识符在搜索操作中传递 -

| 编号 | 参数            | 说明                                                         |
| ---- | --------------- | ------------------------------------------------------------ |
| 1    | Q               | 此参数用于指定查询字符串。                                   |
| 2    | lenient         | 基于格式的错误可以通过将此参数设置为`true`来忽略。默认情况下为`false`。 |
| 3    | fields          | 此参数用于在响应中选择返回字段。                             |
| 4    | sort            | 可以通过使用这个参数获得排序结果，这个参数的可能值是`fieldName`，`fieldName:asc`和`fieldname:desc` |
| 5    | timeout         | 使用此参数限定搜索时间，响应只包含指定时间内的匹配。默认情况下，无超时。 |
| 6    | terminate_after | 可以将响应限制为每个分片的指定数量的文档，当到达这个数量以后，查询将提前终止。 默认情况下不设置`terminate_after`。 |
| 7    |                 | 从命中的索引开始返回。默认值为`0`。                          |
| 8    | size            | 它表示要返回的命中数。默认值为`10`。                         |

#### 5.短语搜索

##### 5.1.短语搜索是ElasticSearch中比较常用的方式,相关的语法个格式为JSON,如下:(这里使用POSTMAN进行演示)

```java
//请求的地址为:
http://192.168.142.131:9200/test/_search
```

请求体:

```java
{
    "query": {
        "match_phrase": {
            "name": "李"
        }
    }
}
```

响应:

```java
{
    "took": 26,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
    },
    "hits": {
        "total": 1,
        "max_score": 0.19178301,
        "hits": [
            {
                "_index": "test",
                "_type": "doc",
                "_id": "AWueHx85GWxHTP0ts5i7",
                "_score": 0.19178301,
                "_source": {
                    "name": "李四",
                    "id": 2,
                    "update_time": "2019-06-13T10:24:55.000Z",
                    "@version": "1",
                    "age": 13,
                    "@timestamp": "2019-06-28T12:46:04.976Z"
                }
            }
        ]
    }
}
```

![ZMsrUH.png](https://s2.ax1x.com/2019/06/28/ZMsrUH.png)

##### 5.2.提高精度搜索:

搜索结果精准控制的第一步：灵活使用and关键字，如果你是希望所有的搜索关键字都要匹配的，那么就用and，可以实现单纯match query无法实现的效果 

```java
{
    "query": {
        "match_phrase": {
            "name": {
         		 "query": "李",
        		 "operator": "and"
            }
        }
    }
}
```

控制搜索结果的精准度的第二步：指定一些关键字中，必须至少匹配其中的多少个关键字，才能作为结果返回  

```java
{
    "query": {
        "match_phrase": {
            "name": {
        		"query": "李",
        		"minimum_should_match": "10%"
            }
        }
    }
}
```

用bool组合多个搜索条件，来搜索name

```java
{
  "query": {
    "bool": {
      "must":     { "match": { "name": "李" }},
      "must_not": { "match": { "name": "张"  }},
      "should": [
                  { "match": { "age": "13" }},
                  { "match": { "name": "四"   }}
      ]
    }
  }
}
```

- 注意:bool组合多个搜索条件，如何计算relevance score

  must和should搜索对应的分数，加起来，除以must和should的总数

  排名第一：test，同时包含should中所有的关键字，testOne，testTwo
  排名第二：test，同时包含should中的testOne
  排名第三：test，不包含should中的任何关键字

  　　should是可以影响相关度分数的，must是确保说，谁必须有这个关键字，同时会根据这个must的条件去计算出document对这个搜索条件的relevance score，在满足must的基础之上，should中的条件，不匹配也可以，但是如果匹配的更多，那么document的relevance score就会更高

- 默认情况下，should是可以不匹配任何一个的，比如上面的搜索中，"李四"，就不匹配任何一个should条件，但是有个例外的情况，如果没有must的话，那么should中必须至少匹配一个才可以，比如下面的搜索，should中有4个条件，默认情况下，只要满足其中一个条件，就可以匹配作为结果返回，但是可以精准控制，should的4个条件中，至少匹配几个才能作为结果返回 

```java
{
  "query": {
    "bool": {
      "should": [
        { "match": { "name": "李四" }},
        { "match": { "name": "张三"   }},
        { "match": { "name": "王五"   }},
        { "match": { "name": "赵六"   }}
      ],
      "minimum_should_match": 3 
    }
  }
}  
```

- 指定返回属性

  只返回查询文档的name和age属性

```java
{
  "query": {
    "bool": {
      "should": [
        { "match": { "name": "李四" }},
        { "match": { "name": "张三"   }},
        { "match": { "name": "王五"   }},
        { "match": { "name": "赵六"   }}
      ],
      "minimum_should_match": 1
    }
  },
  "_source": [
        "name",
        "age"
    ]
}
```

- 高亮搜索

```java
  {
    "query": {
        "bool": {
            "must": {
                "match": {
                    "name": {
                        "query": "李四",
                        "minimum_should_match": "100%"
                    }
                }
            }
        }
    },
    "highlight": {
        "fields": {
            "name": {}
        }
    }
}
```

  返回如下:

```java
{
    "took": 72,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
    },
    "hits": {
        "total": 1,
        "max_score": 0.2712221,
        "hits": [
            {
                "_index": "test",
                "_type": "doc",
                "_id": "AWueHx85GWxHTP0ts5i7",
                "_score": 0.2712221,
                "_source": {
                    "name": "李四",
                    "id": 2,
                    "update_time": "2019-06-13T10:24:55.000Z",
                    "@version": "1",
                    "age": 13,
                    "@timestamp": "2019-06-28T12:46:04.976Z"
                },
                "highlight": {
                    "name": [
                        "<em>李</em><em>四</em>"
                    ]
                }
            }
        ]
    }
}
```

#### 6.使用SpringBoot+ElasticSearch

​	这里我们需要注意SpringBoot和ElasticSearch的版本号能对应,我们查看官网的介绍进行构建项目:

<https://github.com/spring-projects/spring-data-elasticsearch> ,主要点如下所示:

Maven configuration

Add the Maven dependency:

```
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-elasticsearch</artifactId>
    <version>x.y.z.RELEASE</version>
</dependency>
```

If you’d rather like the latest snapshots of the upcoming major version, use our Maven snapshot repository and declare the appropriate dependency version.

```
<dependency>
  <groupId>org.springframework.data</groupId>
  <artifactId>spring-data-elasticsearch</artifactId>
  <version>x.y.z.BUILD-SNAPSHOT</version>
</dependency>

<repository>
  <id>spring-libs-snapshot</id>
  <name>Spring Snapshot Repository</name>
  <url>https://repo.spring.io/libs-snapshot</url>
</repository>
```

##### Versions

The following table shows the Elasticsearch versions that are used by Spring Data Elasticsearch:

| Spring Data Elasticsearch | Elasticsearch |
| ------------------------- | ------------- |
| 3.2.x                     | 6.7.2         |
| 3.1.x                     | 6.2.2         |
| 3.0.x                     | 5.5.0         |
| 2.1.x                     | 2.4.0         |
| 2.0.x                     | 2.2.0         |
| 1.3.x                     | 1.5.2         |

我们这里的elastic的版本为2.4.1,所以我们选取2.4.1.RELEASE:

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.data/spring-data-elasticsearch -->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-elasticsearch</artifactId>
            <version>2.1.14.RELEASE</version>
        </dependency>
```

然后在application.yml配置一下服务器的elasticsearch地址(注意线上的建立连接监听的端口为9300,并非9200)： 

```java
spring:
  data:
    elasticsearch:
      cluster-name: my-application
      cluster-nodes: 服务器Ip:9300
```

配置好后启动我们的项目,会打印如下的日志,说明连接成功:

![ZlCkY8.png](https://s2.ax1x.com/2019/06/29/ZlCkY8.png)

完整pom文件如下:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.13.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.home</groupId>
    <artifactId>es-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>es-demo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.data/spring-data-elasticsearch -->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-elasticsearch</artifactId>
            <version>2.1.14.RELEASE</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

##### 入门Demo:

> ElasticsearchCRUD介绍网站:<https://damienbod.com/2014/10/01/full-text-search-with-asp-net-mvc-jquery-autocomplete-and-elasticsearch/> 
>
> The article explains how to use the ElasticsearchCRUD NuGet package. ElasticsearchCRUD is designed so that you can do CRUD operations for any entity and insert, delete, update or select single documents from Elasticsearch. The package only includes basic search or query possibilities. (本文解释了如何使用ElasticsearchCRUD NuGet包。ElasticsearchCRUD的设计使您可以对任何实体执行CRUD操作，并从Elasticsearch中插入、删除、更新或选择单个文档。该包只包含基本的搜索或查询可能性。 )

##### 6.1.1.我们先基于student表创建一个实体类用于测试

```java
package com.home.esdemo.entity;

import org.springframework.data.elasticsearch.annotations.Document;

import java.util.Date;

/**
 * indexName索引名称，type类别
 */
@Document(indexName = "test",type = "student")
public class Student {

    private  Integer id;
    private  String name;
    private  Integer age;
    private  Date updateTime;

    public Student() {
    }

    public Student(Integer id, String name, Integer age, Date updateTime) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.updateTime = updateTime;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Date getUpdateTime() {
        return updateTime;
    }

    public void setUpdateTime(Date updateTime) {
        this.updateTime = updateTime;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", updateTime=" + updateTime +
                '}';
    }
}

```

##### 6.1.2.新建新建ElasticsearchRepository接口

```java
@Repository
public interface StudentRepository extends ElasticsearchRepository<Student, Integer> {
    Student findAllByName(String name);
}

```

##### 6.1.3.测试

```java
package com.home.esdemo;

import com.home.esdemo.entity.Student;
import com.home.esdemo.esinterface.StudentRepository;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.awt.print.Book;
import java.util.Date;

@RunWith(SpringRunner.class)
@SpringBootTest
public class EsDemoApplicationTests {

    @Autowired
    StudentRepository studentRepository;

    @Test
    public void contextLoads() {
        Student student = new Student(8, "小白", 18, new Date());
        studentRepository.index(student);//保存
        Student name = studentRepository.findAllByName("小"); //查找
        System.out.println(name);
        
        bookRepository.save( new Student(9, "其他", 28, new Date()));
    }

}

}

```

##### StudentRepository可以使用如下关键字来实现复杂的操作:

| 关键字              | 使用示例                           | 等同于的ES查询                                               |
| :------------------ | :--------------------------------- | :----------------------------------------------------------- |
| And                 | findByNameAndPrice                 | {“bool” : {“must” : [ {“field” : {“name” : “?”}}, {“field” : {“price” : “?”}} ]}} |
| Or                  | findByNameOrPrice                  | {“bool” : {“should” : [ {“field” : {“name” : “?”}}, {“field” : {“price” : “?”}} ]}} |
| Is                  | findByName                         | {“bool” : {“must” : {“field” : {“name” : “?”}}}}             |
| Not                 | findByNameNot                      | {“bool” : {“must_not” : {“field” : {“name” : “?”}}}}         |
| Between             | findByPriceBetween                 | {“bool” : {“must” : {“range” : {“price” : {“from” : ?,”to” : ?,”include_lower” : true,”include_upper” : true}}}}} |
| LessThanEqual       | findByPriceLessThan                | {“bool” : {“must” : {“range” : {“price” : {“from” : null,”to” : ?,”include_lower” : true,”include_upper” : true}}}}} |
| GreaterThanEqual    | findByPriceGreaterThan             | {“bool” : {“must” : {“range” : {“price” : {“from” : ?,”to” : null,”include_lower” : true,”include_upper” : true}}}}} |
| Before              | findByPriceBefore                  | {“bool” : {“must” : {“range” : {“price” : {“from” : null,”to” : ?,”include_lower” : true,”include_upper” : true}}}}} |
| After               | findByPriceAfter                   | {“bool” : {“must” : {“range” : {“price” : {“from” : ?,”to” : null,”include_lower” : true,”include_upper” : true}}}}} |
| Like                | findByNameLike                     | {“bool” : {“must” : {“field” : {“name” : {“query” : “? *”,”analyze_wildcard” : true}}}}} |
| StartingWith        | findByNameStartingWith             | {“bool” : {“must” : {“field” : {“name” : {“query” : “? *”,”analyze_wildcard” : true}}}}} |
| EndingWith          | findByNameEndingWith               | {“bool” : {“must” : {“field” : {“name” : {“query” : “*?”,”analyze_wildcard” : true}}}}} |
| Contains/Containing | findByNameContaining               | {“bool” : {“must” : {“field” : {“name” : {“query” : “?”,”analyze_wildcard” : true}}}}} |
| In                  | findByNameIn(Collectionnames)      | {“bool” : {“must” : {“bool” : {“should” : [ {“field” : {“name” : “?”}}, {“field” : {“name” : “?”}} ]}}}} |
| NotIn               | findByNameNotIn(Collectionnames)   | {“bool” : {“must_not” : {“bool” : {“should” : {“field” : {“name” : “?”}}}}}} |
| True                | findByAvailableTrue                | {“bool” : {“must” : {“field” : {“available” : true}}}}       |
| False               | findByAvailableFalse               | {“bool” : {“must” : {“field” : {“available” : false}}}}      |
| OrderBy             | findByAvailableTrueOrderByNameDesc | {“sort” : [{ “name” : {“order” : “desc”} }],”bool” : {“must” : {“field” : {“available” : true}}}} |

