---
title: Zookeeper集群ZKUI可视化界面部署整理文档
categories:
 - Zookeeper
tags:
 - 后端开发
description: server.0、server.1、server.2 为 zk 集群中三个 node 的信息，定义格式为 hostname:port1:port2，其中 port1 是 node 间通信使用的端口，port2 是node 选举使用的端口，需确保三台主机的这两个端口都是互通的...
---

#### 1.环境说明

```java
JDK     		1.8 +    【不要安装OpenJdk】
zookeeper   	3.4.14
zkUI  			2.0.+
maven      	    3.5.+
```

#### 2.源码安装

##### 1.1、获取tar包

```java
http://apache.spinellicreations.com/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz
```

##### 1.2、上传tar包,解压改名

```java
# 解压文件
tar -xvf zookeeper-3.4.14.tar.gz

# 改名
mv zookeeper-3.4.14 zookeeper

# 删除无用的tar包
rm -f zookeeper-3.4.14.tar.gz
```

##### 1.3、修改zookeeper配置

```java
# 进入到 zookeeper/conf 下面
cd zookeeper/conf

# 将 zoo_sample.cfg 复制一份，命名为 zoo.cfg，此即为Zookeeper的配置文件
cp zoo_sample.cfg zoo.cfg

# 编辑配置文件
vim zoo.cfg
```

默认配置如下:

![t03qQs.png](https://s1.ax1x.com/2020/06/04/t03qQs.png) 

修改配置如下：

 ```java
# The number of milliseconds of each tick

tickTime=2000

# The number of ticks that the initial 

# synchronization phase can take

initLimit=10

# The number of ticks that can pass between 

# sending a request and getting an acknowledgement

syncLimit=5

# the directory where the snapshot is stored.

# do not use /tmp for storage, /tmp here is just 

# example sakes.

 
#数据存放位置

dataDir=/data/zookeeper/data

 
#日志存放的位置

dataLogDir=/data/log/zookeeper/logs
 

# the port at which the clients will connect

clientPort=2181

# the maximum number of client connections.

# increase this if you need to handle more clients

#maxClientCnxns=60

#

# Be sure to read the maintenance section of the 

# administrator guide before turning on autopurge.

#

# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance

#

# The number of snapshots to retain in dataDir

#autopurge.snapRetainCount=3

# Purge task interval in hours

# Set to "0" to disable auto purge feature

#autopurge.purgeInterval=1

 
#集群的配置,后面为你的ip,后面有节点依次往下面排


server.0=172.31.3.86:2888:3888

server.1=172.31.3.87:2888:3888

Server.2=172.31.3.26:2888:3888

# The number of milliseconds of each tick

tickTime=2000

# The number of ticks that the initial 

# synchronization phase can take

initLimit=10

# The number of ticks that can pass between 

# sending a request and getting an acknowledgement

syncLimit=5

# the directory where the snapshot is stored.

# do not use /tmp for storage, /tmp here is just 

# example sakes.

 

#数据存放位置

dataDir=/data/zookeeper/data

 

#日志存放的位置

dataLogDir=/data/log/zookeeper/logs

 

# the port at which the clients will connect

clientPort=2181

# the maximum number of client connections.

# increase this if you need to handle more clients

#maxClientCnxns=60

#

# Be sure to read the maintenance section of the 

# administrator guide before turning on autopurge.

#

# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance

#

# The number of snapshots to retain in dataDir

#autopurge.snapRetainCount=3

# Purge task interval in hours

# Set to "0" to disable auto purge feature

#autopurge.purgeInterval=1

 

#集群的配置,后面为你的ip,后面有节点依次往下面排

 
server.0=172.31.3.86:2888:3888

server.1=172.31.3.87:2888:3888

server.2=172.31.3.26:2888:3888

 ```

最后使用wq!保存修改,说明:

dataDir 和 dataLogDir 需要在启动前创建完成

clientPort 为 zookeeper的服务端口

server.0、server.1、server.2 为 zk 集群中三个 node 的信息，定义格式为 hostname:port1:port2，其中 port1 是 node 间通信使用的端口，port2 是node 选举使用的端口，需确保三台主机的这两个端口都是互通的。

##### 1.4、更改日志配置

```java
Zookeeper 默认会将控制台信息输出到启动路径下的 zookeeper.out 中，通过如下方法，可以让 Zookeeper 输出按尺寸切分的日志文件：

修改 conf/log4j.properties 文件，将

      zookeeper.root.logger=INFO, CONSOLE

   改为

     zookeeper.root.logger=INFO, ROLLINGFILE

修改bin/zkEnv.sh文件，将

      ZOO_LOG4J_PROP="INFO,CONSOLE"

  改为

     ZOO_LOG4J_PROP="INFO,ROLLINGFILE"

```

![t03OLq.png](https://s1.ax1x.com/2020/06/04/t03OLq.png)

![t08ufe.png](https://s1.ax1x.com/2020/06/04/t08ufe.png)



 

##### 1.5、同上操作，在另外的机器也是如此操作



##### 1.6、创建 myid 文件

```java
切换到 zookeeper 目录下

# zookeeper 存放数据目录

    mkdir -p /data/zookeeper/data 

    cd /data/zookeeper/data

   touch myid 

# 这里的 0 和  zoo.cfg配置文件server.x  对应上面配置集群节点的数字,比如第一台就写入数字0即可:

# 此处为第一台服务器的示例,其余的依次改为1、2

    echo "0" > myid 

分别在三台主机的 dataDir 路径下创建一个文件名为 myid 的文件，文件内容为该 zk 节点的编号

```



##### 1.7、启动服务，默认后台启动

```java
分别启动配置的 zookeeper 服务

cd /data/zookeeper/bin

./zkServer.sh start

```

返回信息：

![t08G0P.png](https://s1.ax1x.com/2020/06/04/t08G0P.png)

##### 1.8、查看集群状态

节点启动完成后，可依次执行如下命令查看集群状态：

```java
 ./zkServer.sh status
```

可以看到三台已经选举成功:
![t08dpQ.png](https://s1.ax1x.com/2020/06/04/t08dpQ.png)

![t08BXn.png](https://s1.ax1x.com/2020/06/04/t08BXn.png) 
![t08gtU.png](https://s1.ax1x.com/2020/06/04/t08gtU.png)

 

# ZkUI部署文档

#### **1.** 源码安装

##### 1.1、下载源码

```java
# ZKUI2.0的GitHub地址:

https://github.com/DeemOpen/zkui

```

##### 1.2、Maven打包

```java
# 进入下载的ZKUI2.0的项目文件夹,执行:

mvn install -Dmaven.test.skip=true

```

![t08X1H.png](https://s1.ax1x.com/2020/06/04/t08X1H.png)


![t08znI.png](https://s1.ax1x.com/2020/06/04/t08znI.png) 

 

打包成功后在target目录下可以看到如下zkui-2.0-SNAPSHOT-jar-with-dependencies.jar,这个正是我们需要的jar包

![t0GPN8.png](https://s1.ax1x.com/2020/06/04/t0GPN8.png) 

##### **1.3、**上传jar包和配置文件cfg到服务器

配置文件位置在项目文件的根目录下,如下图所示:

![t0Geun.png](https://s1.ax1x.com/2020/06/04/t0Geun.png) 

```java
#创建存放目录 放入上面的jar包和cfg文件  一起放在/data/zkui目录下

mkdir /data/zkui

```

#### **2.** 修改配置

```java
修改conf.cfg配置文件，将zookeeper地址，多个zk实例被逗号分开。例如：

172.31.3.86:2181,172.31.3.87:2181,172.31.3.26:2181

```

![t0G1CF.png](https://s1.ax1x.com/2020/06/04/t0G1CF.png) 

#### **3.** 启动服务

放入启动脚本zkui_start.sh:

```java
#!/bin/bash

. /etc/profile

 

ROOT_PATH=$(dirname $(readlink -f $0))

#获取可执行jar包名称

function getJar() {

 for item in $(ls $ROOT_PATH); do

  fileName=$item

  if [ ! -d $fileName ]; then

   if [ ${fileName##*.} = jar ]; then

    APP_NAME=$fileName

    return

   fi

  fi

 done

 echo "当前目录下未找到可执行jar包"

 exit 1

}

getJar

 

 

JAVA_OPTS="-server \

     -Xms1024m \

     -Xmx1024m \

     -Xmn512m \

     -Xss512m \

     -XX:MaxMetaspaceSize=512m \

     -XX:MaxNewSize=512m \

     -XX:MetaspaceSize=512m \

     -XX:SurvivorRatio=9 \

     -XX:+UseConcMarkSweepGC \

     -XX:ParallelCMSThreads=4 \

     -XX:MaxTenuringThreshold=4 \

     -XX:+UseCMSCompactAtFullCollection \

     -XX:CMSFullGCsBeforeCompaction=4 \

     -XX:+CMSParallelRemarkEnabled \

     -XX:+CMSScavengeBeforeRemark \

     -XX:+CMSClassUnloadingEnabled \

     -XX:+UseCMSInitiatingOccupancyOnly \

     -XX:CMSInitiatingOccupancyFraction=80 \

     -XX:CMSInitiatingPermOccupancyFraction=92 \

     -XX:SoftRefLRUPolicyMSPerMB=0 \

     -XX:+UseFastAccessorMethods \

     -XX:+AggressiveOpts \

     -XX:-UseBiasedLocking \

     -XX:+DisableExplicitGC"

 

#使用说明，用来提示输入参数

usage() {

 echo "Usage: sh 执行脚本.sh [start|stop|restart|status]"

 exit 1

}

 

#检查程序是否在运行

is_exist() {

 pid=$(ps -ef | grep $APP_NAME | grep -v grep | awk '{print $2}')

 #如果不存在返回1，存在返回0

 if [ -z "${pid}" ]; then

  return 1

 else

  return 0

 fi

}

 

#启动方法

start() {

 is_exist

 if [ $? -eq "0" ]; then

  echo "${APP_NAME} is already running. pid=${pid} ."

 else

  nohup java ${JAVA_OPTS} -jar ${ROOT_PATH}/${APP_NAME} >/dev/null 2>&1 &

  #nohup java $JAVA_OPTS -jar $APP_NAME >/www/log.txt 2>&1 &

 fi

}

 

#停止方法

stop() {

 is_exist

 if [ $? -eq "0" ]; then

  kill -9 $pid

 else

  echo "${APP_NAME} is not running"

 fi

}

 

#输出运行状态

status() {

 is_exist

 if [ $? -eq "0" ]; then

  echo "${APP_NAME} is running. Pid is ${pid}"

 else

  echo "${APP_NAME} is NOT running."

 fi

}

 

#重启

restart() {

 stop

 start

}

 

#根据输入参数，选择执行对应方法，不输入则执行使用说明

case "$1" in

"start")

 start

 ;;

"stop")

 stop

 ;;

"status")

 status

 ;;

"restart")

 restart

 ;;

*)

 usage

 ;;

esac

```

使用脚本启动:

```java
sh zkui_start.sh start
```

停止命令参考:

```java
sh zkui_start.sh stop
```

检测是否启动成功:

```java
netstat -antp |grep 9090
```

#### **4.** 界面相关

启动完成后访问:

http://192.168.2.128:9090/

输入我们配置中的账密,默认是:

```java
admin | manager
```

![t0GrgH.png](https://s1.ax1x.com/2020/06/04/t0GrgH.png) 

查看节点信息界面:

![t0G4PS.png](https://s1.ax1x.com/2020/06/04/t0G4PS.png) 

查看集群状态:

![t0Gz24.png](https://s1.ax1x.com/2020/06/04/t0Gz24.png) 

 

#### **5.** 配置开机自启

##### 1.1、在 /lib/systemd/system/ 目录下创建 zookeeper.service  的配置文件

```java
vim zookeeper.service 
```

 zookeeper.service 添加内容：

 ```java
[Unit]

Description=Zookeeper service

After=network.target

 

[Service]

Type=simple

Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/java/jdk-11.0.1/bin"

User=root

Group=root

ExecStart=/opt/kafka/kafka_2.11-2.1.0/bin/zookeeper-server-start.sh /opt/kafka/kafka_2.11-2.1.0/config/zookeeper.properties

ExecStop=/opt/kafka/kafka_2.11-2.1.0/bin/zookeeper-server-stop.sh

Restart=on-failure

 

[Install]

WantedBy=multi-user.target

 ```

注：以上文件 根据自己的 jdk 和 kafka 安装目录相应的修改。

##### 1.2、刷新配置

```java
 systemctl daemon-reload
```

##### 1.3、zookeeper服务加入开机自启

```java
systemctl enable zookeeper
```

##### 1.4、使用systemctl启动/关闭/重启 zookeeper服务

```java
systemctl start/stop/restart  zookeeper 
```

##### 1.5、查看状态

```java
systemctl status zookeeper
```





 



