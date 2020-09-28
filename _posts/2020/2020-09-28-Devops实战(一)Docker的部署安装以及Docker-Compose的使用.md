---
title: Devops实战(一)Docker的部署安装以及Docker-Compose的使用
categories:
 - Devops 
tags:
 - 后端开发
description: Docker是一组平台即服务（PaaS）产品，它们使用操作系统级虚拟化以称为容器的软件包交付软件。容器彼此隔离，并将它们自己的软件，库和配置文件捆绑在一起；他们可以通过定义明确的渠道相互交流。所有容器都由单个操作系统内核运行，因此使用的资源少于虚拟机...
---

#### 1.docker和docker-Compose简介

		Docker是一组平台即服务（PaaS）产品，它们使用操作系统级虚拟化以称为容器的软件包交付软件。容器彼此隔离，并将它们自己的软件，库和配置文件捆绑在一起；他们可以通过定义明确的渠道相互交流。所有容器都由单个操作系统内核运行，因此使用的资源少于虚拟机。
	
		Compose 是一个用户定义和运行多个容器的 Docker 应用程序。在 Compose 中你可以使用 YAML 文件来配置你的应用服务。然后，只需要一个简单的命令，就可以创建并启动你配置的所有服务。

#### 2.安装docker和docker-Compose

##### docker的安装

- 阿里云

```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

- daocloud

```sh
 curl -sSL https://get.daocloud.io/docker | sh
```

安装后将会自动重启

docker的卸载

```
sudo apt-get remove docker docker-engine
rm -fr /var/lib/docker/
```

配置加速器(以下是本人阿里云加速配置)

```java
mkdir -p /etc/docker
touch /etc/docker/daemon.json
vim /etc/docker/daemon.json

{"registry-mirrors":["https://asmtpu24.mirror.aliyuncs.com"]}


systemctl daemon-reload
systemctl restart docker
```

##### docker-Compose的安装

可以通过修改 URL 中的版本，自定义您需要的版本。

- Github源

```
curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

- Daocloud镜像

```sh
curl -L https://get.daocloud.io/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

卸载

```
sudo rm /usr/local/bin/docker-compose
```

使用

①新建 docker-compose.yml 文件

通过以下配置，在运行后可以创建两个站点(只为演示)

```yml
version: "2"
services:
  test:
    hostname: test
    image: tomcat:8
    volumes:
      - "./target/test.war:/usr/local/tomcat/webapps/test.war"
    ports:
      - "38000:8080"
    entrypoint:
      - "catalina.sh"
      - "run"
```

此处只是简单演示写法，说明 docker-compose 的方便

②构建完成，后台运行镜像

```
docker-compose up -d
```

运行后就可以使用 ip+port 访问这两个站点了

③镜像更新重新部署

```
docker-compose down
docker-compose pull
docker-compose up -d
```



