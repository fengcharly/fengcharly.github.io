---
title: Devops实战(三)Kubenets与minikube的安装以及使用实战
categories:
 - Devops 
tags:
 - 后端开发
description: Kubernetes（常简称为K8s）是用于自动部署、扩展和管理「容器化（containerized）应用程序」的开源系统。该系统由Google设计并捐赠给Cloud Native Computing Foundation（今属Linux基金会）来使用。它旨在提供“跨主机集群的自动部署、扩展以及运行应用程序容器的平台”。它支持一系列容器工具, 包括Docker等...
---

#### 1.Kubenets与minikube简介

		Kubernetes（常简称为K8s）是用于自动部署、扩展和管理「容器化（containerized）应用程序」的开源系统。该系统由Google设计并捐赠给Cloud Native Computing Foundation（今属Linux基金会）来使用。它旨在提供“跨主机集群的自动部署、扩展以及运行应用程序容器的平台”。它支持一系列容器工具, 包括Docker等。
	
		Minikube 是一种可以让您在本地轻松运行 Kubernetes 的工具。Minikube 在笔记本电脑上的虚拟机（VM）中运行单节Kubernetes 集群，供那些希望尝试 Kubernetes 或进行日常开发的用户使用。

#### 2.使用centos7安装

首先确保你已经安装过Docker，没有安装可以参考前面的docker安装步骤进行安装docker。

##### ① 下载阿里的 minikube和kubectl

```java
# curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.4.0/minikube-linux-amd64

# curl -Lo kubectl    http://kubernetes.oss-cn-hangzhou.aliyuncs.com/kubernetes-release/release/v1.16.0/bin/linux/amd64/kubectl
```

##### ② 加执行权限并放入启动目录

```java
 # chmod +x minikube

 # chmod +x kubectl
     
copy到/usr/local/bin

 # cp minikube /usr/local/bin

 # cp kubectl /usr/local/bin 
     
开启kubelet服务,这一步很重要    
     
 systemctl enable kubelet.service
```

##### ③启动minikube, 并发布一个tomcat

 ```java
// 没有使用virtualBox, 没用kvm2。none用的是本地的docker
# minikube start --vm-driver=none   
    
// 手工下载tomcat 8.0 image
# docker pull tomcat:8.0          

// 部署服务
# kubectl create deployment tomcat --image=tomcat:8.0

// 暴露对外访问端口
# kubectl expose deployment tomcat --type=NodePort --port=8080

// 使用minikube启动pod    
# minikube service tomcat  
 ```

启动完后会输出如下所示:

![dEJatO.png](https://s1.ax1x.com/2020/08/16/dEJatO.png)

 

然后访问URL的地址http://192.168.17.129:30743，会弹出tomcat的界面如下所示：

![dEJ7Bq.png](https://s1.ax1x.com/2020/08/16/dEJ7Bq.png)

##### 3.访问minikube的图形化界面

minikube还给我们准备了图形化的界面，可通过如下命令开启界面：

```java
minikube dashboard
```

同样我们可以在启动日志中找到相应的地址,但是只能本机访问,如果需要外部访问可以按照如下方式设置代理:

- 添加集群对外访问代理：

```java
nohub kubectl proxy  --port=[需要暴露的端口号] --address='[服务器IP]' --accept-hosts='^[外部访问服务器的IP]$'  >/dev/null 2>&1& 
```

- 例如:

```java
nohup kubectl proxy  --port=8088 --address='192.168.17.129' --accept-hosts='^192.168.17.129$'  >/dev/null 2>&1& 
```

访问地址示例:

```java
http://192.168.17.129:8088/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/overview?namespace=default
```

进入后可以看到如下界面,这个界面方便我们管理kubectl的集群已经部署的服务,关于minikube的命令的详细介绍可以参考[官方文档中的Minikube 功能小节](https://kubernetes.io/zh/docs/setup/learning-environment/minikube/):

![dEUG8O.png](https://s1.ax1x.com/2020/08/16/dEUG8O.png)









