---
title: "使用certbot申请证书"
date: "2023-07-30T04:28:22.412Z"
draft: 
category: [] 
tags: [centos,日常]
series: []
cover: 
    image: 'https://certbot.eff.org/assets/Certbot-solid-c4e500f9953fc8ee1d38cb0b22778163602a82cb2b39a5bc89211315c5c877c9.svg'
    alt: 'source https://certbot.eff.org/'
    caption: 'source https://certbot.eff.org/'
---
官方推荐使用snap安装  
Unless you have very specific requirements, we kindly suggest that you use the installation instructions for your system found at  [https://certbot.eff.org/instructions](https://certbot.eff.org/instructions).    

![image.png](https://image.jysgdyc.top:443/blog/20230730124707.png)  

```shell
# 查看系统版本
cat /etc/centos-release

# 安装snapd
yum install snapd

# 安装后，需要启用管理snap通信套接字的systemd unit
systemctl enable --now snapd.socket

# 为了启用classic snap的支持，需要创建如下软连接
ln -s /var/lib/snapd/snap /snap

# 重启后
snap install core
snap refresh core

# 安装certbot
snap install --classic certbot

ln -s /snap/bin/certbot /usr/bin/certbot

# 验证方式
# https://letsencrypt.org/zh-cn/docs/challenge-types/
# 建议使用dns txt记录验证
# https://eff-certbot.readthedocs.io/en/stable/using.html#manual

certbot --manual certonly --preferred-challenges=dns

# 根据certbot提示，会让输入用户名与邮箱
# 然后输入需要解析的域名，certbot就会输出一个txt内容
# 此时需要把这个内容放到指定的txt解析域名下（一般是_acme-challenge）
# 查看txt域名解析，解析值对上certbot输出的内容后，再下一步
dig TXT <yourdomain.com>

# 查看证书
certbot certificates


# 续订
certbot renew


```

