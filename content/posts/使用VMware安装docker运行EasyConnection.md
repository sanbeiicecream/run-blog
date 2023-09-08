---
title: "VMware使用docker安装EasyConnection"
date: "2023-04-22T16:15:01.099Z"
draft: 
category: []
tags: [日常, docker]
series: []
cover: 
    image: ''
    alt: ''
    caption: ''
---

公司内网的Jenkins需要使用EasyConnection才能访问，但是这软件有点流氓  
在自己电脑上是未开源的代理软件一概不敢使用怕有后门，所以在git上找到一个可以使用docker容器运行这软件的库  
https://github.com/Hagb/docker-easyconnect  
由于我的windows系统，现在可以在wsl2环境上进行试验，但是为了方便我在VMWare的centos系统上进行了操作  
果然过于复杂的东西都不是能够一下就成功的  
### docker安装
```shell
yum remove docker  docker-common docker-selinux docker-engine

# 安装需要的软件包， yum-util 提供yum-config-manager功能，另两个是devicemapper驱动依赖
yum install -y yum-utils device-mapper-persistent-data lvm2
# 设置一个yum源，下面两个都可用
yum-config-manager --add-repo http://download.docker.com/linux/centos/docker-ce.repo（中央仓库）
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo（阿里仓库）
# 选择docker版本并安装 （1）查看可用版本有哪些
yum list docker-ce --showduplicates | sort -r
#（2）选择一个版本并安装：yum install docker-ce-版本号
yum -y install docker-ce-20.10.9

systemctl start docker
systemctl enable docker
```

### 启动容器
作者提供了两个版本镜像，一个是纯cli的，一个提供了vnc界面  
由于是使用centos所以直接使用cli版本的
```shell
docker run --device /dev/net/tun --cap-add NET_ADMIN -ti -p 127.0.0.1:1080:1080 -p 127.0.0.1:8888:8888 -e EC_VER=7.6.3 -e CLI_OPTS="-d vpnaddress -u username -p password" --name ec hagb/docker-easyconnect:cli
```
+ vpnaddress -> 代理的地址
+ username -> 密码
+ password -> 账号

运行后可以看到打印  
![image.png](https://image.jysgdyc.top:443/blog-images/20230423101919.png)  
然后在本地ping虚拟机上的端口，居然不通
![image.png](https://image.jysgdyc.top:443/blog-images/20230423103230.png)  
虚拟机上1080 与 8888这两个端口，防火墙也是放行的  
经过一段时间的尝试，弄不出来，于是在Issue里面寻找答案，经过一番寻找，原来是命令里面的端口映射的方式有问题  
https://github.com/Hagb/docker-easyconnect/issues/103#issuecomment-1016429807  
**把127.0.0.1去掉就可以了**  

---
### 为什么加上ip就不行了呢
首先是命令的差异性   
`docker run  -p  ip:hostPort:containerPort`  
`hostPort:containerPort`（映射所有接口地址）  
`ip:hostPort:containerPort` （映射指定地址的指定端口）  
加上127.0.0.1是映射127.0.0.1:port到docker的port上，而由于是使用的虚拟机，还需要先连上虚拟机的ipv4地址，所以是**127.0.0.1 != 虚拟机ip**导致的  
如果非想加上ip的话，有两个办法可以实现
1. 在虚拟机里面使用有转发功能的软件指定一个端口转发到127.0.0.1:8888上（nginx，caddy）
2. 直接使用ipv4地址

---
然后在浏览器里面使用Proxy SwitchyOmega扩展添加一下虚拟机的centos机器的8888/http代理或者1080/socket5代理  
直接访问需要通过easyconnetion连接的路径  
![image.png](https://image.jysgdyc.top:443/blog-images/20230423111639.png)
成功进入！
