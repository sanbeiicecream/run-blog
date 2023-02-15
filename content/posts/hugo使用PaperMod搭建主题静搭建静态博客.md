---
title: "hugo使用PaperMod搭建主题静搭建静态博客"
date: 2023-02-16T02:51:14+08:00
draft: false
---

## 先动起来
配置优先的一个静态页面生成的框架
先安装hugo后，需要先安装golang
``` shell
   # 先创建一个模板
   hugo new site xxx
   # 然后下载一个主题放在themes文件夹里面，在config.yml中theme指定主题（文件夹名）
   # 启动查看效果
   hugo server
```
下载主题最好是使用git submodule命令进行仓库链接，这样后续可以对主题的修改进行单独维护
``` shell
  git init
  git submodule add https://github.com/adityatelange/hugo-PaperMod/
```
![](https://image.jysgdyc.top:443/blog-images/2023/02/16/20230216024613.png)
fork的原因在于当原始的仓库进行变更后，可以选择性进行合并，以便即使现在版本落后很多也不会合并和样式出现大范围问题。
进行自定义时，不能修改themes里面的主题，因为是通过git submodule add的，修改后，推代码并不会推送到github上，要么修改自己fork的主题库，**最好是通过在外面通过和theme一样的层级文件进行覆盖**
初级改造，先改造得顺眼，后面再加各种功能
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

