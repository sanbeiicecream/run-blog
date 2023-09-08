---
title: "Docker容器之间网络通信"
date: "2023-07-14T01:14:34.575Z"
draft: 
category: [] 
tags: [Docker]
series: []
cover: 
    image: ''
    alt: ''
    caption: ''
---
https://docs.docker.com/network/  
```shell
# 查看docker网络
docker network ls 
```
1. 在创建容器时，使用-P -p 可以端口映射与宿主机进行通信；如果多个容器使用同一个主机端口创建容器会报错
   ![image.png](https://image.jysgdyc.top:443/blog/20230714224309.png)

2. 容器之间可以直接通过ip地址通信，可以使用`docker inspect 【name】| grep IPAddress `查看容器ip地址。这种方式不太好，需要先获取地址，才能设置docker启动命令；
3. **使用主机网络模式**：在运行容器时，可以使用`--network=host`标志，将容器加入宿主机的网络命名空间。这样，容器将直接使用宿主机的网络接口，可以直接通过宿主机的IP地址进行通信。但请注意，这将使容器与宿主机共享网络栈，可能导致端口冲突和安全隐患。
4. **通过docker的桥接网络 bridge**：相当于将多个容器添加进一个网络里面，使用内部的==dns==进行解析访问
```shell
# 创建一个网络环境，名为networkName:
docker network create networkName 

# 分别将两个容器添加到这个网络中
docker network connect networkName 【容器名】
docker network connect networkName 【容器名】

# 访问地址：【容器名】:8080
```
