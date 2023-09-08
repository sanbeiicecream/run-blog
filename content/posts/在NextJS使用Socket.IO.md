---
title: "在NextJS使用Socket.IO"
date: "2023-09-04T03:49:45.927Z"
draft: 
category: [] 
tags: [Web, NextJS, Socket.IO]
series: []
cover: 
    image: ''
    alt: ''
    caption: ''
---

由于socketio是借用nextjs启动的服务进行创建的，在app router下无法在端口处获取到完整的req与res  
目前的nextJS版本（13.3+）
```ts
export async function GET(request: Request, res: Response) {
  console.log("🚀 ~ file: route.ts:3 ~ GET ~ res:", res)
  return new Response('Hello, Next.js!')
}
```

可以看见使用app router时，response对象并不会在这里进行完整返回，而是在nextJS内部  
![image.png](https://image.jysgdyc.top:443/blog/20230904115458.png)

那么就需要使用**page router**，处理socket连接，使用page router可以获取到完整的req与res  
![image.png](https://image.jysgdyc.top:443/blog/20230904115535.png)

## WebSocket
[wiki](https://zh.wikipedia.org/wiki/WebSocket)  
WebSocket是一种网络传输协议，可在单个TCP连接上进行全双工通信，位于OSI模型的应用层。  
WebSocket使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。在WebSocket API中，浏览器和服务器只需要完成一次握手，两者之间就可以建立持久性的连接，并进行双向数据传输。  
为了实现兼容性，WebSocket握手使用HTTP Upgrade从HTTP协议更改为WebSocket协议。  
默认情况下，Websocket协议使用80端口；运行在TLS之上时，默认使用443端口。  
### 握手协议
WebSocket 是独立的、建立在TCP上的协议。  
Websocket 通过 HTTP/1.1 协议的101状态码进行握手。  
为了建立Websocket连接，需要通过浏览器发出请求，之后服务器进行回应，这个过程通常称为“握手”（Handshaking）。  
一个典型的Websocket握手请求如下  
客户端请求：  
```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Origin: http://example.com
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```

服务器回应：  
```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
```

**字段说明**
Connection必须设置Upgrade，表示客户端希望连接升级。  
Upgrade字段必须设置Websocket，表示希望升级到Websocket协议。  
Sec-WebSocket-Key是随机的字符串，服务器端会用这些数据来构造出一个SHA-1的信息摘要。把“Sec-WebSocket-Key”加上一个特殊字符串“258EAFA5-E914-47DA-95CA-C5AB0DC85B11”，然后计算SHA-1摘要，之后进行Base64编码，将结果做为“Sec-WebSocket-Accept”头的值，返回给客户端。如此操作，可以尽量避免普通HTTP请求被误认为Websocket协议。  
Sec-WebSocket-Version 表示支持的Websocket版本。RFC6455要求使用的版本是13，之前草案的版本均应当弃用。  
Origin字段是必须的。如果缺少origin字段，WebSocket服务器需要回复HTTP 403 状态码（禁止访问）。  
其他一些定义在HTTP协议中的字段，如Cookie等，也可以在Websocket中使用。  


可以看出握手协议的时候需要计算出Sec-WebSocket-Accept的值，所以一般是直接使用第三方库


## Socket.IO
[官网](https://socket.io/zh-CN/)  

![image.png](https://image.jysgdyc.top:443/blog/20230904115848.png)

Socket.IO在服务器端有多种构造方式，在这里由于是需要借用nextJS启动的服务所有使用`new Server(httpServer[, options])`的方式  
里面的httpServer可以使用上面req或res里面的socket对象下的server  
![image.png](https://image.jysgdyc.top:443/blog/20230904115922.png)

创建后再将实例存到socket对象上，下次请求就直接判断有没有这个值，有的话就说明当前请求里已经进行了WebSocket连接，直接返回  
```js
if (res.socket?.server?.io) {
  return;
}

const io = new ServerIO(res.socket.server as any, {
	path: <path>,
	addTrailingSlash: false  // 这里不加的话，path后面路径会多个斜杠
	});

io.on('connection', socket => {
  // something...
});

res.socket.server.io = io;
```

在创建ServerIO实例时，如果绑定了server，并配置了path，那么下次请求这个路径，Socket.io 会自动处理 WebSocket 握手，如果不是WebSocket握手而是其他方法则会返回  
![image.png](https://image.jysgdyc.top:443/blog/20230904120048.png)

## 其他
客户端做了降级处理，如果websocket连接失败，那么就使用长轮询模拟
https://socket.io/zh-CN/docs/v4/client-options/#transports
```js
{
  transports: ["polling", "websocket"]
}
```

socketio默认的握手连接顺序与websocket的不同，socketio的101握手不是直接就进行的，而是需要通过3个轮询请求后，再发起，当101握手成功后，就过一段时间再发起关闭轮询的请求，完成整套操作一共需要请求5次  

![image.png](https://image.jysgdyc.top:443/blog/20230904120154.png)

1. 握手 (包含会话 ID — 此处会返回一个sid：lx4TP...AAG — 用于后续请求)
2. 发送数据 (HTTP 长轮询)
3. 接收数据 (HTTP 长轮询)
4. 升级 (WebSocket)
5. 接收数据 (HTTP 长轮询, WebSocket连接建立成功后关闭)

连接时[心跳机制](https://socket.io/zh-CN/docs/v4/engine-io-protocol/#protocol)


## 问题
在NextJS版本大于**13.2.5**时会出现连接不上的问题  
![image.png](https://image.jysgdyc.top:443/blog/20230904120300.png)

一直尝试ws握手失败  
![image.png](https://image.jysgdyc.top:443/blog/20230904120314.png)
经过查询发现可以使用自定义服务启动时进行设置  
[https://stackoverflow.com/questions/76277148/using-socket-io-with-next-js](https://stackoverflow.com/questions/76277148/using-socket-io-with-next-js)
可是还是不行，虽然这个握手操作能返回101但是会直接断开连接  
发现是nextjs14+的版本对于没有对[upgrade请求的监听](https://github.com/vercel/next.js/issues/49334#issuecomment-1545054961)导致这个ws握手一直失败  
官方提供了一个[Draft](https://github.com/vercel/next.js/pull/54502)  
```js
const upgradeHandler = app.getUpgradeHandler()

server.on("upgrade", async (req, socket, head) => {
  await upgradeHandler(req, socket, head)
})
```

截止当前最新版本（13.4.20-canary.15），**还没有用！！！**


但是发现个现象，第一次连接会失败，然后重启服务，不刷新页面，居然就可以了  
![image.png](https://image.jysgdyc.top:443/blog/20230904120503.png)

只要一刷新页面就不行了



## 总结

当前NextJS与Socket.IO开发还有缺陷，无法使用WebSocket进行通信，只能使用轮询方式进行通信  
**暂时就使用轮询进行开发**











