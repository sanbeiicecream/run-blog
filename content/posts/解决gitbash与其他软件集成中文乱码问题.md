---
title: "解决gitbash与其他软件集成中文乱码问题"
date: "2023-03-13T18:19:19.890Z"
draft: 
category: [] 
tags: [工具]
series: []
cover: 
    image: 'https://image.jysgdyc.top:443/blog-images/20230314022101.png'
    alt: ''
    caption: ''
---

**结论：使用其他工具集成gitbash如果不加载配置会出现中文乱码问题**  

先在git bash中将字符编码改成UTF-8  

![image.png](https://image.jysgdyc.top:443/blog-images/20230314022159.png)
![image.png](https://image.jysgdyc.top:443/blog-images/20230314022222.png)

然后在其他集成git bash的软件中添加**-i -l** 参数加载git bash的配置文件  
### win11 终端  
在终端的JSON配置中修改一下属性值
```json
{
	...
	"commandline": "%PROGRAMFILES%\\bin\\bash.exe -i -l",
	"icon": "%PROGRAMFILES%\\mingw64\\share\\git\\git-for-windows.ico",
}
```
**%PROGRAMFILES% 代表git bash安装目录**  

## vscode
```json
{
	"terminal.integrated.profiles.windows": {
	"gitBash": {
	  "path": "%PROGRAMFILES%\\bin\\bash.exe",
	  "args": ["-i", "-l"]
	}
	},
	"terminal.integrated.defaultProfile.windows": "gitBash",
}
```

### 打开外部终端
这时就没有参数可以像上面那样直接设置了，但是可以直接打开win11的终端，让win11的终端默认开启一个git bash，从而间接创建一个git bash。
```json
{
  "terminal.explorerKind": "external",
  "terminal.external.windowsExec": "%LocalAppData%\\Microsoft\\WindowsApps\\wt.exe",
}
```