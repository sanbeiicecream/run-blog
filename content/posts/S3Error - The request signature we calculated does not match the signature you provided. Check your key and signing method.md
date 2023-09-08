---
title: "S3Error - The request signature we calculated does not match the signature you provided. Check your key and signing method"
date: "2023-06-28T02:36:52.162Z"
draft: 
category: [] 
tags: [minio]
series: []
cover: 
    image: ''
    alt: ''
    caption: ''
---

在使用[minio-js](https://github.com/minio/minio-js) sdk上传文件时，出现  
==S3Error: The request signature we calculated does not match the signature you provided. Check your key and signing method.==  

![image.png](https://image.jysgdyc.top:443/blog/20230628103135.png)  

是不是服务器时区不对导致的？通过设置时间同步
```shell
# 时区设置
sudo timedatectl set-timezone Asia/Shanghai

# 安装NTP（Network Time Protocol）服务，它用于同步系统时间
sudo yum install ntp

# 启动
systemctl start ntp
systemctl enable ntp

date
```
发现还是会出现这个问题  
经过不断尝试发现查看桶数据的API是可以的打印出信息的，那么问题可能出现在上传接口，由于后台是用nginx转发的，通过查看nginx日志发现上传的时候出现了一个HEAD请求  

![image.png](https://image.jysgdyc.top:443/blog/20230628103329.png)

当请求路径出现两个杠==//==时会使用HEAD请求头返回403就会提示上面哪个报错  
发现是在使用原来是使用putObject api时，传入的对象名称在前面多加了一个==/==去掉即可  
然后又报了：==S3Error: Access Denied.==  
发现是请求链接上添加了请求参数location，找不到就报错了  

![image.png](https://image.jysgdyc.top:443/blog/20230628103616.png)

需要给账号设置桶权限"s3: GetBucketLocation"    
