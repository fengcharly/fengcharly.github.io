---
title: Devops实战(二)了解Dockerfile与私有仓库Harbor的构建
categories:
 - Devops 
tags:
 - 后端开发
description: Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明...
---

#### 1.什么是Dockerfile

Dockerfile 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。

![](https://s1.ax1x.com/2020/08/16/dEUqsJ.jpg)

#### 2.Dockerfile指令详解

##### COPY

复制指令，从上下文目录中复制文件或者目录到容器里指定路径。

格式：

```
COPY [--chown=<user>:<group>] <源路径1>...  <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",...  "<目标路径>"]
```

**[--chown=<user>:<group>]**：可选参数，用户改变复制到容器内文件的拥有者和属组。

**<源路径>**：源文件或者源目录，这里可以是通配符表达式，其通配符规则要满足 Go 的 filepath.Match 规则。例如：

```
COPY hom* /mydir/
COPY hom?.txt /mydir/
```

**<目标路径>**：容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建。

##### ADD

ADD 指令和 COPY 的使用格式一致（同样需求下，官方推荐使用 COPY）。功能也类似，不同之处如下：

- ADD 的优点：在执行 <源文件> 为 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，会自动复制并解压到 <目标路径>。
- ADD 的缺点：在不解压的前提下，无法复制 tar 压缩文件。会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。具体是否使用，可以根据是否需要自动解压来决定。

##### CMD

类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:

- CMD 在docker run 时运行。
- RUN 是在 docker build。

**作用**：为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束。CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖。

**注意**：如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效。

格式：

```
CMD <shell 命令> 
CMD ["<可执行文件或命令>","<param1>","<param2>",...] 
CMD ["<param1>","<param2>",...]  # 该写法是为 ENTRYPOINT 指令指定的程序提供默认参数
```

推荐使用第二种格式，执行过程比较明确。第一种格式实际上在运行的过程中也会自动转换成第二种格式运行，并且默认可执行文件是 sh。

##### ENTRYPOINT

类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。

但是, 如果运行 docker run 时使用了 --entrypoint 选项，此选项的参数可当作要运行的程序覆盖 ENTRYPOINT 指令指定的程序。

**优点**：在执行 docker run 的时候可以指定 ENTRYPOINT 运行所需的参数。

**注意**：如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效。

格式：

```
ENTRYPOINT ["<executeable>","<param1>","<param2>",...]
```

可以搭配 CMD 命令使用：一般是变参才会使用 CMD ，这里的 CMD 等于是在给 ENTRYPOINT 传参，以下示例会提到。

示例：

假设已通过 Dockerfile 构建了 nginx:test 镜像：

```
FROM nginx

ENTRYPOINT ["nginx", "-c"] # 定参
CMD ["/etc/nginx/nginx.conf"] # 变参 
```

1、不传参运行

```
$ docker run  nginx:test
```

容器内会默认运行以下命令，启动主进程。

```
nginx -c /etc/nginx/nginx.conf
```

2、传参运行

```
$ docker run  nginx:test -c /etc/nginx/new.conf
```

容器内会默认运行以下命令，启动主进程(/etc/nginx/new.conf:假设容器内已有此文件)

```
nginx -c /etc/nginx/new.conf
```

##### ENV

设置环境变量，定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。

格式：

```
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```

以下示例设置 NODE_VERSION = 7.2.0 ， 在后续的指令中可以通过 $NODE_VERSION 引用：

```
ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc"
```

##### ARG

构建参数，与 ENV 作用一至。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效，也就是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量。

构建命令 docker build 中可以用 --build-arg <参数名>=<值> 来覆盖。

格式：

```
ARG <参数名>[=<默认值>]
```

##### VOLUME

定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。

作用：

- 避免重要的数据，因容器重启而丢失，这是非常致命的。
- 避免容器不断变大。

格式：

```
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
```

在启动容器 docker run 的时候，我们可以通过 -v 参数修改挂载点。

##### EXPOSE

仅仅只是声明端口。

作用：

- 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。
- 在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。

格式：

```
EXPOSE <端口1> [<端口2>...]
```

##### WORKDIR

指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。（WORKDIR 指定的工作目录，必须是提前创建好的）。

docker build 构建镜像过程中的，每一个 RUN 命令都是新建的一层。只有通过 WORKDIR 创建的目录才会一直存在。

格式：

```
WORKDIR <工作目录路径>
```

##### USER

用于指定执行后续命令的用户和用户组，这边只是切换后续命令执行的用户（用户和用户组必须提前已经存在）。

格式：

```
USER <用户名>[:<用户组>]
```

##### HEALTHCHECK

用于指定某个程序或者指令来监控 docker 容器服务的运行状态。

格式：

```
HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

HEALTHCHECK [选项] CMD <命令> : 这边 CMD 后面跟随的命令使用，可以参考 CMD 的用法。
```

##### ONBUILD

用于延迟构建命令的执行。简单的说，就是 Dockerfile 里用 ONBUILD 指定的命令，在本次构建镜像的过程中不会执行（假设镜像为 test-build）。当有新的 Dockerfile 使用了之前构建的镜像 FROM test-build ，这是执行新镜像的 Dockerfile 构建时候，会执行 test-build 的 Dockerfile 里的 ONBUILD 指定的命令。

格式：

```
ONBUILD <其它指令>
```

#### 3.使用Dockerfile构建镜像实战

##### ①创建Dockerfile:

```java
touch Dockerfile

// 创建nginx的目录 下一步会用到
mkdir -p /run/nginx
```

##### ②编辑vim Dockerfile:

```java
FROM nginx:1.14.0

RUN mkdir -p /usr/local/nginx/html  && mkdir -p /data/wwwlogs && chown nginx. /data/wwwlogs -R && apt-get update && apt-get install -y curl wget telnet vim procps unzip
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

##### ③构建镜像

```java
docker build -t nginx-1.14 .
```

在 Dockerfile 目录下执行以上命令即可构建镜像。`-t` 参数指定了镜像名称为 `nginx-alpine`，最后的 `.` 表示构建上下文（`.` 表示当前目录）.

##### ④查看并启动镜像

```java
// 查看
docker images
    
// 启动
docker run --name my-nginx -p 80:80 -d nginx-1.14
```

访问相应的IP可以看到nginx的界面:

![dE0rwj.png](https://s1.ax1x.com/2020/08/16/dE0rwj.png)

#### 4.安装私有仓库Harbor

​		随着docker的发展,一款私有化的docker镜像仓库随之诞生,它私有化的部署很好的保证了安全性，它就是Harbor。Harbor是由VMware公司开源的企业级的Docker Registry管理项目，它包括权限管理（RBAC）、LDAP、日志审核、界面管理、自我注册、镜像复制和中文支持等功能。

##### 下载安装包：

```java
# 离线安装包
wget https://github.com/vmware/harbor/releases/download/v1.1.2/harbor-offline-installer-v1.1.2.tgz
tar zxvf harbor-offline-installer-v1.1.2.tgz
    
# 在线安装包
wget https://github.com/vmware/harbor/releases/download/v1.1.2/harbor-online-installer-v1.1.2.tgz
tar zxvf harbor-online-installer-v1.1.2.tgz
```

##### 配置Harbor：

```java
# cd harbor
# vi harbor.conf
 
## Configuration file of Harbor
 
# hostname设置访问地址，可以使用ip、域名，不可以设置为127.0.0.1或localhost
hostname = 你的主机IP
 
# 访问协议，默认是http，也可以设置https，如果设置https，则nginx ssl需要设置on
ui_url_protocol = http
 
# mysql数据库root用户默认密码root123，实际使用时修改下
db_password = root123
 
max_job_workers = 3
customize_crt = on
ssl_cert = /data/cert/server.crt
ssl_cert_key = /data/cert/server.key
secretkey_path = /data
admiral_url = NA
 
# 邮件设置，发送重置密码邮件时使用
email_identity =
email_server = smtp.mydomain.com
email_server_port = 25
email_username = sample_admin@mydomain.com
email_password = abc
email_from = admin <sample_admin@mydomain.com>
email_ssl = false
 
# 启动Harbor后，管理员UI登录的密码，默认是Harbor12345
harbor_admin_password = Harbor12345
 
# 认证方式，这里支持多种认证方式，如LADP、本次存储、数据库认证。默认是db_auth，mysql数据库认证
auth_mode = db_auth
 
# LDAP认证时配置项
ldap_url = ldaps://ldap.mydomain.com
#ldap_searchdn = uid=searchuser,ou=people,dc=mydomain,dc=com
#ldap_search_pwd = password
ldap_basedn = ou=people,dc=mydomain,dc=com
#ldap_filter = (objectClass=person)
ldap_uid = uid
ldap_scope = 3
ldap_timeout = 5
 
# 是否开启自注册
self_registration = on
 
# Token有效时间，默认30分钟
token_expiration = 30
 
# 用户创建项目权限控制，默认是everyone（所有人），也可以设置为adminonly（只能管理员）
project_creation_restriction = everyone
 
verify_remote_cert = on

```

修改完配置文件后，在当前目录执行./install.sh，Harbor服务就会根据当前目录下的docker-compose.yml开始下载依赖的镜像，检测并按照顺序依次启动各个服务。

默认的登录地址和密码：

```java
http://你的虚拟机IP/harbor/sign-in

账号：admin
密码: Harbor12345
```

![dErbhn.png](https://s1.ax1x.com/2020/08/16/dErbhn.png)

![dErXcV.png](https://s1.ax1x.com/2020/08/16/dErXcV.png)

#### 5.推送镜像到Harbor

这里我建了一个项目名为home，然后将我们之前打包的镜像nginx-1.14推送到Harbor仓库

##### ①解决https的问题:

```java
// 这里我们建立的是http的链接,需要编辑放行IP
vim /etc/docker/daemon.json
    
// 加入如下内容 主要是 "insecure-registries":["192.168.1.103"]
{
    "insecure-registries":["192.168.1.103"]
}

systemctl daemon-reload
systemctl restart docker
```

##### ③使用Docker登录Harbor仓库:

```java
docker login http://192.168.1.103/

账号：admin
密码: Harbor12345 
```

##### ④推送镜像:

```java
// 标记镜像
docker tag nginx-1.14 192.168.1.103/home/mynginx:v1

// 推送镜像到当前项目
docker push 192.168.1.103/home/mynginx:v1
    
// 推送成功即可在仓库看到该镜像,拉取命令如下:
docker pull 192.168.1.103/home/mynginx:v1
```





