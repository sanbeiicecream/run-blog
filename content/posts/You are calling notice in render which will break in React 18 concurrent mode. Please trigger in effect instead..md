---
title: "You are calling notice in render which will break in React 18 concurrent mode. Please trigger in effect instead."
date: "2023-03-05T18:47:39.922Z"
draft: 
category: [] 
tags: [前端,React,Antd]
series: []
cover: 
    image: 'https://image.jysgdyc.top:443/blog-images/20230306025136.png'
    alt: ''
    caption: ''
---
最近把以前的项目依赖升级一下（React升级到了18.x， antd升级到了5.x），以便看一下新的API更新了什么，在使用antd的message时候发现有时不会显示。 在控制台发现了上面那个提示。   
看这个提示应该是antd里面的提示，于是在antd的message组件源码中，找到了这行
```jsx
if (!holderRef.current) {
        process.env.NODE_ENV !== "production" ? warning(false, 'Message', 'You are calling notice in render which will break in React 18 concurrent mode. Please trigger in effect instead.') : void 0;
        const fakeResult = () => {};
        fakeResult.then = () => {};
        return fakeResult;
}
```
意思是没有 这个golderRef才会提示这个问题  
发现在显示message时会在根目录下创建一个div元素，提醒完后就会移除这个div
![image.png](https://image.jysgdyc.top:443/blog-images/20230306172246.png)


![image.png](https://image.jysgdyc.top:443/blog-images/20230306210456.png)
