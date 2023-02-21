---
title: "使用obsidian的Templater插件替代hugo new命令"
date: "2023-02-20T11:59:58.421Z"
draft: 
category: [] 
tags: ['建站']
series: []
cover: 
    image: ''
    alt: ''
    caption: ''
---

使用obsidian进行博客文章编辑时，需要下载一个生成模板的工具，不然使用hugo new xxx生成模板有点麻烦，需要下载一个插件
![image.png](https://image.jysgdyc.top:443/blog-images/20230220195704.png)
![image.png](https://image.jysgdyc.top:443/blog-images/20230220195740.png)

[官方配置](https://silentvoid13.github.io/Templater/internal-functions/internal-modules/date-module.html)
``` md
  ---
  title: "<% tp.file.title %>"
  date: "<% tp.date.now() %>"
  draft: false
  tags:
  ---
```
然后使用obsidian创建一个新文件，插入模板
有是有了但是不够友好，时间应该生成ISO8601的格式
![image.png](https://image.jysgdyc.top:443/blog-images/20230220195818.png)

修改插件代码
![image.png](https://image.jysgdyc.top:443/blog-images/20230220195836.png)

找到插件位置，发现这插件是用JS写的
![image.png](https://image.jysgdyc.top:443/blog-images/20230220195849.png)

找到main.js中关键代码片段
![image.png](https://image.jysgdyc.top:443/blog-images/20230220195907.png)

加这几行代码，完成！！！easy
![image.png](https://image.jysgdyc.top:443/blog-images/20230220195921.png)

在模板中`date: "<% tp.date.now('iso') %>"`
重启obsidian，试一试是ok的
![image.png](https://image.jysgdyc.top:443/blog-images/20230220195937.png)
