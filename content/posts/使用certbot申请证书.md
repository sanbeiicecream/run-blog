---
title: 使用certbot申请证书
date: 2023-07-30T04:28:22.412Z
last: 2023-09-08 01:03:06
draft: 
category: []
tags:
  - centos
  - 日常
series: []
cover:
  image: https://certbot.eff.org/assets/Certbot-solid-c4e500f9953fc8ee1d38cb0b22778163602a82cb2b39a5bc89211315c5c877c9.svg
  alt: source https://certbot.eff.org/
  caption: source https://certbot.eff.org/
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


# 常规续订
certbot renew

# 删除证书
certbot delete --cert-name <example.com>
# or to choose from a list:
certbot delete

```

## 额外续订操作
使用 `--manual` 创建的证书不支持自动续订，除非通过 `--manual-auth-hook` 与身份验证挂钩脚本结合使用以自动设置所需的 HTTP 和/或 TXT 质询。  
![image.png](https://image.jysgdyc.top:443/blog/20230907230212.png)

[详细文档](https://eff-certbot.readthedocs.io/en/stable/using.html#)  
强行使用cerbot renew 会报错  
![image.png](https://image.jysgdyc.top:443/blog/20230907224420.png)



由于是通过外部信息进行验证，所以需要添加验证身份的脚本验证成功后才能续订  
[Renewal with the manual plugin](https://eff-certbot.readthedocs.io/en/stable/using.html#manual-renewal)  

```
# 
certbot certonly --manual --manual-auth-hook /path/to/http/authenticator.sh --manual-cleanup-hook /path/to/http/cleanup.sh -d secure.example.com
```

具体例子可以看官网  
[Pre and Post Validation Hooks](https://eff-certbot.readthedocs.io/en/stable/using.html#pre-and-post-validation-hooks)  

本质上这种方法相当于把手动续订自动化了，通过API创建**DNS Record**，后续使用certbot renew 就会去执行这个脚本，然后更新（类似于续订）  
如果域名注册网站没有提供操作DNS Record的API那么也是不行的。
