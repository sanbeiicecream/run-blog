---
title: 使用Vant组件时报错
date: 2023-10-27T06:58:26.640Z
last: 
draft: 
category: []
tags: []
series: []
cover:
  image: ""
  alt: ""
  caption: ""
---

在使用Vant二次封装表单组件时，发现控制台报了个错
Swipe.mjs:183  Uncaught (in promise) TypeError: Cannot read properties of null (reading 'offsetWidth')  
![image.png](https://image.jysgdyc.top:443/blog/20231020094509.png)

根据页面点击，发现当页面出现输入框与下拉框时在两个不同组件点击时会触发该报错  
触发步骤：下拉框弹出选项后 -> 输入框 -> 下拉框

## 调试Swipe文件
由于vite是利用es6的模块进行的js加载，所以可以看见打包成mjs的代码，方便调试  
**修改文件后，需要先删除node_modules下的.vite缓存**  
![image.png](https://image.jysgdyc.top:443/blog/20231020094659.png)

Vant打包将jsx语法转换成js，并且es6 -> es5  
![image.png](https://image.jysgdyc.top:443/blog/20231020094725.png)

这个组件调用链，最外层组件是PickerGroup  
![image.png](https://image.jysgdyc.top:443/blog/20231027145233.png)


发现代码中确实是使用到了这个组件  
当需要弹出框时，根据点击的表单项类型展示不同组件（**picker**， **datePicker**，**pickerGroup**）  
```vue
<van-popup>
  <van-picker v-if='xxx'/>
  <van-date-picker v-else-if='xxx'/>
  <van-picker-group v-else/>
</van-popup>
```

通过调试Swipe.js文件  
![image.png](https://image.jysgdyc.top:443/blog/20231027145550.png)

分析触发步骤，模态框出现后，插槽内的组件会被渲染；模态框隐藏后，插槽内组件并**没有被销毁**；模态框中插槽根据点击类型显示（**Picker**，**datePicker**，**pickerGroup**）；由于pickerGroup组件是使用的v-else进行判断，点击其他表单项（例如输入框），类型无法匹配picker，dataPicker，所以就初始化了pickerGroup；此时再次点击其他类型（**Picker**，**datePicker**），此时pickerGroup组件中Swipe下监听器会被触发，但是因为此时模态框中不是pickerGroup组件，所以报错。  
![image.png](https://image.jysgdyc.top:443/blog/20231027145654.png)


## 解决办法
将pickerGroup放在单独popup组件中




