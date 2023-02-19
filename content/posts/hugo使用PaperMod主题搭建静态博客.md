---
title: "hugo使用PaperMod搭建主题静搭建静态博客"
date: 2023-02-16T02:51:14+08:00
draft: 
category: 
tags: ['建站']
series: 
---

## 先动起来
配置优先的一个静态页面生成的框架
先安装hugo后，需要先安装golang
``` shell
   # 先创建一个模板
   hugo new site xxx
   # 然后下载一个主题放在themes文件夹里面，在config.yml中theme指定主题（文件夹名）
```
下载主题最好是使用git submodule命令进行仓库链接，这样后续可以对主题的修改进行单独维护

``` shell
  # 先进入模板文件夹
  cd xxx
  # 初始化git
  git init
  # 关联主题子项目
  git submodule add https://github.com/adityatelange/hugo-PaperMod/ themes/PaperMod
```
**Why?**
![](https://image.jysgdyc.top:443/blog-images/2023/02/16/20230216024613.png)
fork的原因在于当原始的仓库进行变更后，可以选择性进行合并，以便即使现在版本落后很多也不会合并和样式出现大范围问题。
进行自定义时，不能修改themes里面的主题，因为是通过git submodule add的，修改后，推代码并不会推送到github上，要么修改自己fork的主题库，**最好是通过在外面通过和theme一样的层级文件进行覆盖**
### 修改配置
配置文件在当前模板的根下，但是hugo默认创建的是toml格式的配置文件，这个格式的个人感觉没有yml格式的好用，从PaperMod的exampleSite分支拷贝yml配置过来[hugo-PaperMod/config.yml at exampleSite · adityatelange/hugo-PaperMod (github.com)](https://github.com/adityatelange/hugo-PaperMod/blob/exampleSite/config.yml)
修改
```yml
# 这个值为themes文件夹主题的名称，用哪个就写哪个
theme: PaperMod
```
然后运行看效果
```shell
   # 启动查看效果
   hugo server
```
![image.png](https://image.jysgdyc.top:443/blog-images/2023/02/18/20230218001029.png)

动起来了！！！

## 初级改造
先改造得顺眼，后面再加各种功能
![](https://image.jysgdyc.top:443/blog-images/2023/02/16/20230216025423.png)
有些配置修改后需要重启服务才会生效，比如i18n里面的配置
根据页面类名定位到这里是在partial目录下的post_meta.html中定义的模板
![](https://image.jysgdyc.top:443/blog-images/2023/02/16/20230216025450.png)
这三个标签需要在content里面添加archive.md search.md才能显示出来
``` serach.md
  ---
  title: "🔍搜索" # in any language you want
  layout: "search" # is necessary
  # url: "/archive"
  # description: "Description for Search"
  summary: "search"
  placeholder: "输入关键字 + Enter"
  ---
```

``` archive.md
  ---
  title: "⏱时间轴"
  layout: "archives"
  url: "/archives/"
  summary: archives
  ---
```


## 配置详解
为什么把这个放最后，因为这配置项有很多，本来主题都是用的现成的，要定制一下只需要几个关键的属性就够了，这配置都当个速查手册了。