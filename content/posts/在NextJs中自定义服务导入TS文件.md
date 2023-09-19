---
title: 在NextJs中自定义服务导入TS文件
date: 2023-09-19T18:07:51.159Z
last: 
draft: false
category: []
tags: []
series: []
cover:
  image: ""
  alt: ""
  caption: ""
---
在项目中一般不应该TS与JS混合开发，但是也有一些例外，有些依赖包没有使用TS开发，所以TS提供了一个选项`"allowJs": true`  
设置了就能马上运行了吗？还不行！
这个选项只是让你在ts中引入js文件不会报错而已  
如果不加这个选项，在导入js文件时会提示  
![image.png](https://image.jysgdyc.top:443/blog/20230920024040.png)

nextJS项目是用TS开发的，如果单就一个自定义服务使用JS编写也没什么，但是在这个JS模块中中需要引入其他TS的模块，这就有点麻烦了，所以自定义服务需要用TS  
## 模块类型
首先要解决的就是模块化的问题  
![image.png](https://image.jysgdyc.top:443/blog/20230920024112.png)

在node项目中默认的js文件是使用**CommonJS**模块规范（使用 `require` 和 `module.exports` 来定义和导出模块）所以会报错  
解决办法两个  
1. 在package.json里面添加`"type": "module"`可以解决  
2. 或者使用`node --experimental-modules module.js`  
如果有些文件必须要用CommonJS模块，比如一些配置文件postcss、next，那么可以把后缀换成cjs，这样模块系统就会把这些文件看成CommonJS模块  

## 使用TS
当直接用Node运行TS文件时会报错  
![image.png](https://image.jysgdyc.top:443/blog/20230920024230.png)

那么如何使用Node运行TS文件呢？  

### 使用ts-node
ts-node是将ts文件编译成js然后执行的一个命令工具  
还是提示解析不了这个后缀  
![image.png](https://image.jysgdyc.top:443/blog/20230920024254.png)

发现原来是ts-node**默认**只支持**CommonJS**模块  
https://github.com/TypeStrong/ts-node/issues/1007  
在回答中找到可以解决的一个办法  
添加一个配置项--esm  
```shell
ts-node --esm 
```

也可以在tsconfig.json里添加  
```json
"ts-node": {
  "esm": true
}
```

但是还是会报错  
这次是不能找到模块了  
![image.png](https://image.jysgdyc.top:443/blog/20230920024409.png)

发现是在使用 ES 模块时我们需要在导入语句中指定文件扩展名，平时使用CLI工具创建的项目不需要的原因是因为webpack之类的工具已经帮我们处理好了  

https://stackoverflow.com/questions/65384754/error-err-module-not-found-cannot-find-module  

添加上js后缀后（不需要用ts后缀，因为ts需要经过转换成js文件），成功启动  
![image.png](https://image.jysgdyc.top:443/blog/20230920024447.png)


### 一个现象

```typescript
// a.ts
const a = 1
const b = '2'

export default { a, b }
```

![image.png](https://image.jysgdyc.top:443/blog/20230920024535.png)

当导入使用别名时，居然会ts会认为这个是一个CommonJS模块，需要用default获取值  
发现是tsconfig.json的有两个配置有关  
```json
{
  "moduleResolution": "Node",
  // 与上面的值是有关联的，当为node时才能为true
  "resolveJsonModule": true
}

```

moduleResolution有两个值：`node`、`classic`  
用于控制 TypeScript 在解析模块导入语句时的行为。具体来说，它定义了 TypeScript 如何查找和解析模块的过程  
1. node：这是默认值。当 `moduleResolution` 设置为 `"node"` 时，TypeScript 使用 Node.js 模块解析规则  
2. classic：TypeScript 使用传统的模块解析规则。这种解析方式更类似于传统的浏览器环境  

`esModuleInterop`
用于控制 TypeScript 如何处理导入和导出模块时的默认行为
当设置为 `true` 时，TypeScript 将启用 ES6 模块系统的导入和导出行为，同时允许你导入 CommonJS 模块的默认导出
当设置为 `false` 时，TypeScript 将使用传统的 CommonJS 导入和导出行为，不允许直接导入 CommonJS 模块的默认导出
访问 CommonJS 模块的默认导出时要使用 `.default` 属性
