---
title: Devops实战(四)Rancher的部署与安装详解
categories:
 - Devops 
tags:
 - 后端开发
description: Rancher是一个开源的企业级容器管理平台。通过Rancher，企业再也不必自己使用一系列的开源软件去从头搭建容器服务平台。Rancher提供了在生产环境中使用的管理Docker和Kubernetes的全栈化容器部署与管理平台...
---

#### Rancher简介

Rancher是一个开源的企业级容器管理平台。通过Rancher，企业再也不必自己使用一系列的开源软件去从头搭建容器服务平台。Rancher提供了在生产环境中使用的管理Docker和Kubernetes的全栈化容器部署与管理平台。

#### 安装Rancher2.4.5

Rancher的GitHub地址如下,可以找到相应的release:

```java
https://github.com/rancher/rancher/releases
```

##### ①找到如下的文件并下载上传到服务器

![dLMaOs.png](https://s1.ax1x.com/2020/08/31/dLMaOs.png)



##### ②在上传目录下执行以下命令打包成镜像

```java
// 赋予权限
chmod +x rancher-save-images.sh
    
// 开始拉取镜像 由于是国外服务器 是比较慢
./rancher-save-images.sh --image-list ./rancher-images.txt
```

![dLMUyj.png](https://s1.ax1x.com/2020/08/31/dLMUyj.png)

##### ③打包好后的目录结构如下

[![dX6tgS.png](https://s1.ax1x.com/2020/08/31/dX6tgS.png)](https://imgchr.com/i/dX6tgS)

##### ④上传到内网服务器

```java
// 给与权限
chmod +x rancher-load-images.sh
    
// 登录docker仓库
docker login REGISTRY.YOURDOMAIN.COM:PORT -u 用户名 -p 密码

//上传到仓库 执行这行命令的时候，等待的时间有点长，请耐心等待 
./rancher-load-images.sh --image-list ./rancher-images.txt --registry <REGISTRY.YOURDOMAIN.COM:PORT>
```

解决https错误:

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

##### ⑤部署Rancher

```java
docker run -d --restart=unless-stopped \
 
-p 80:80 -p 443:443 \
 
-e CATTLE_SYSTEM_DEFAULT_REGISTRY=<REGISTRY.YOURDOMAIN.COM:PORT> \ # 设置默认的系统镜像仓库
 
-e CATTLE_SYSTEM_CATALOG=bundled \ # 自v2.3.0可用，使用内嵌的 Rancher system charts
 
<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher:<RANCHER_VERSION_TAG>
```

示例:

```java
docker run -d --restart=unless-stopped -p 80:80 -p 443:443 -e CATTLE_SYSTEM_DEFAULT_REGISTRY=192.168.1.103 -e CATTLE_SYSTEM_CATALOG=bundled 192.168.1.103/rancher/rancher/rancher:v2.4.5-rc10
```

##### ⑥启动后访问主机名即可见到首页,设置下密码进入

[![dji5Cj.png](https://s1.ax1x.com/2020/08/31/dji5Cj.png)](https://imgchr.com/i/dji5Cj)

###### 6.1. 设置仓库

点击"Settings" -> "system-default-registry"设置默认的镜像仓库地址

[![dxzYeU.png](https://s1.ax1x.com/2020/09/01/dxzYeU.png)](https://imgchr.com/i/dxzYeU)

[![dxzRFH.png](https://s1.ax1x.com/2020/09/01/dxzRFH.png)](https://imgchr.com/i/dxzRFH)

###### 6.2. 添加集群

选择集群 - > 添加集群 -> 自定义

[![djiqbT.png](https://s1.ax1x.com/2020/08/31/djiqbT.png)](https://imgchr.com/i/djiqbT)

设置名称,然后Kubernetes的版本选用的是1.16.13,注意这里可以选定镜像仓库的地址,如果有区分命名空间需要改为 IP:HOST/空间名.

[![dzCSXt.png](https://s1.ax1x.com/2020/09/01/dzCSXt.png)](https://imgchr.com/i/dzCSXt)

[![dzCdHK.png](https://s1.ax1x.com/2020/09/01/dzCdHK.png)](https://imgchr.com/i/dzCdHK)

其余使用默认配置即可,单击下一步,勾选Etcd和Control,复制最底部的命令到虚拟机:

[![dzPNGQ.png](https://s1.ax1x.com/2020/09/01/dzPNGQ.png)](https://imgchr.com/i/dzPNGQ)

报错解决:

```java
// ①报rke-tools:v0.1.59找不到主要原因是推到私库的rke-tools版本为v0.1.58而拉取的版本为v0.1.59,需要我们手动拉取Psuh上去
docker pull rancher/rke-tools:v0.1.59
docker tag rancher/rke-tools:v0.1.59 192.168.1.103/rancher/rancher/rke-tools:v0.1.59
docker push 192.168.1.103/rancher/rancher/rke-tools:v0.1.59

// ②报 No such image: 192.168.1.103/rancher/rancher/hyperkube:v1.18.6-rancher1
docker pull rancher/hyperkube:v1.17.9-rancher1
docker tag rancher/hyperkube:v1.17.9-rancher1 192.168.1.103/rancher/rancher/hyperkube:v1.17.9-rancher1
docker push 192.168.1.103/rancher/rancher/hyperkube:v1.17.9-rancher1
```

##### ⑦配置Kubectl客户端(如果是2.4.5版本可以不配置这步)

下载相关的tar包

```java
wget https://storage.googleapis.com/kubernetes-release/release/v1.16.13/kubernetes-client-linux-amd64.tar.gz
```

解压并在根目录创建".kube"目录

```java
// 解压
tar -xvf kubernetes-client-linux-amd64.tar.gz

//创建.kube目录
mkdir -p /root/.kube

// 创建config文件
touch /root/.kube/config 
```

编辑config,将rancher中的文件复制到里面即可

[![w9MOWd.png](https://s1.ax1x.com/2020/09/02/w9MOWd.png)](https://imgchr.com/i/w9MOWd)

rancher处于运行状态后如下所示

[![w9m6Cn.png](https://s1.ax1x.com/2020/09/02/w9m6Cn.png)](https://imgchr.com/i/w9m6Cn)

##### ⑧部署服务

我们进入到Rancher的集群eos-test主机中,选择Default,点击部署服务

[![w9lV4e.png](https://s1.ax1x.com/2020/09/02/w9lV4e.png)](https://imgchr.com/i/w9lV4e)

部署我们预先存放好的tomcat的镜像,如下

[![w9l5VK.png](https://s1.ax1x.com/2020/09/02/w9l5VK.png)](https://imgchr.com/i/w9l5VK)](https://imgchr.com/i/w9lQDP)

需要设置最小的内存空间,"高级设置" -> "安全/主机设置"如下

[![w91P2j.png](https://s1.ax1x.com/2020/09/02/w91P2j.png)](https://imgchr.com/i/w91P2j)

[![w91MRJ.png](https://s1.ax1x.com/2020/09/02/w91MRJ.png)](https://imgchr.com/i/w91MRJ)

报错:

```java
ReplicaSet "nginx-5bddc7d447" has timed out progressing.; Deployment does not have minimum availability.
```

出现这种问题是因为只部署了单台主机,无法构成集群,解决办法:

```java
至少两台主机,一台master,一台worker
建议服务器配置 2核4G
```









