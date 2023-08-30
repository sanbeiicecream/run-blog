---
title: "使用echarts-for-react报错：Cannot read properties of undefined (reading 'disconnect')"
date: "2023-08-30T08:14:35.153Z"
draft: 
category: [] 
tags: [Web]
series: []
cover: 
    image: ''
    alt: ''
    caption: ''
---

**结论：ECharts版本太高可能导致这个报错**

[源码地址](https://github.com/hustcc/echarts-for-react)  

![image.png](https://image.jysgdyc.top:443/blog/20230830161540.png)

```jsx
import ReactEcharts from 'echarts-for-react'

<ReactEcharts
  option={}
  style={{ height: '350px', width: '100%' }}
/>
```
依赖  
```json
"dependencies": {
  ...
  "size-sensor": "^1.0.0"  // DOM 元素尺寸监听器，当元素尺寸变化的时候，将会触发回调函数！
},
```
查看函数调用栈  
![image.png](https://image.jysgdyc.top:443/blog/20230830162332.png)

## 调试代码
resizeObserver.js  
在使用`ResizeObserver` 对象的dsconnect方法时提示的错误  
![image.png](https://image.jysgdyc.top:443/blog/20230830162405.png)

可以看见在执行这里时sensor等于undefined，意思是执行到destroy的时候还没有执行bind，所以导致的错误，继续往上层找  
![image.png](https://image.jysgdyc.top:443/blog/20230830162428.png)

sensorPool里面调用的destroy，但是没有看到bind，继续往上层找  
![image.png](https://image.jysgdyc.top:443/blog/20230830162443.png)

在上一层的index文件里面找到导出bind的地方，看是哪里引用的，继续往外层找  
![image.png](https://image.jysgdyc.top:443/blog/20230830162600.png)

在core.js文件里面看见导入的index  
![image.png](https://image.jysgdyc.top:443/blog/20230830162634.png)


## 分析

可以知道在EchartsReact组件卸载的时候会调用dispose，这时会判断有没有echartsElement，有的话就调用clear方法  
这个echartsElement是从哪里来的呢？  
![image.png](https://image.jysgdyc.top:443/blog/20230830162657.png)

![image.png](https://image.jysgdyc.top:443/blog/20230830162708.png)

```js
_this.dispose = function () {
  if (_this.echartsElement) {
    try {
      (0, _sizeSensor.clear)(_this.echartsElement);
    } catch (e) {
      console.warn(e);
    }
    // dispose echarts instance
    _this.echartsLib.dispose(_this.echartsElement);
  }
};


```

在组件render的时候产生的  
![image.png](https://image.jysgdyc.top:443/blog/20230830162756.png)


这里去查看源码
知道了echartsLib是[echarts](https://echarts.apache.org/zh/index.html)导出  
```js
// src/index.js
import echarts from 'echarts';
import EchartsReactCore from './core';

// export the Component the echarts Object.
export default class EchartsReact extends EchartsReactCore {
  constructor(props) {
    super(props);
    this.echartsLib = echarts;
  }
}
```

经过调试发现下面这行执行完后触发了componentWillUnmount  
![image.png](https://image.jysgdyc.top:443/blog/20230830162843.png)

![image.png](https://image.jysgdyc.top:443/blog/20230830162858.png)


然后执行了dispose()，相当于还没渲染完成就卸载了

为什么还没渲染完成就卸载组件了？这就是导致报错的原因  
既然是在echats获取实例的时候报错了，这很大程度可能是因为echats内部导致的报错  
查看echarts-for-react的packagse.json包  
![image.png](https://image.jysgdyc.top:443/blog/20230830162943.png)

发现echarts-for-react 2.x版本只支持到4.x版本  
但是在本地项目中安装了5.x版本  
![image.png](https://image.jysgdyc.top:443/blog/20230830163005.png)

**版本冲突导致的问题！**

在npm install的时候也提示了  
![image.png](https://image.jysgdyc.top:443/blog/20230830163128.png)


## 总结

发现项目内部依赖报错时，需要先检查一下依赖包依赖的外部包版本（peerDependencies）是否安装正确  

如果有sourcemap可以很好看见打包没有压缩的代码，直接在浏览器上进行断点调试更为方便  













