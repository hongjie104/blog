---
layout: post
title: 在react-native中加入socket.io
category: react-native
comments: true
---

想要在react-native框架中实现长链接就得用上socket，react-native自带了一个webSocket，使用方法很简单，我们服务器端使用的是koa框架，配上socket.io简直叼炸天，网上也查过一些资料，都说到socket.io是支持webSocket的，于是乎我就兴高采烈地在react-native上写上了webSocket连接后端的代码，但是一运行。。。尼玛，报错啊，大致的意思就是连不上服务器啊，然后我就用egret引擎中的webSocket在web上也实现了一个客户端，结果也是无法连接。。。太坑爹啊。继续查资料，发现socket.io不支持ws协议。。。好吧，那么我们就在react-native上也使用socket.io吧，这会没问题了吧。

首先安装socket.io，因为react-native作为客户端，所以只要socket.io-client就行了。

执行安装命令：

```
npm install socket.io-client --save
```

然后写上代码:

```
import io from "socket.io-client/socket.io";
// 初始化socket
socket = io('http://77.100.10.19:3001', {jsonp: false});
// 发送消息
socket.emit("test", {"hello": "world"});
// 接收消息
socket.on('test', function (data) {
	console.log("收到消息:", data);
});

```

嗯，react-native客户端红屏报错了，报错的地方是node_modules\socket.io-client\socket.io.js的2985行，原因是socket.io都是用在web上的，而在移动端上navigator.userAgent是个null，那么改一下代码：


```
// var isAndroid = navigator.userAgent.match(/Android/i);
var isAndroid = /Android/i.test(navigator.userAgent);
```

然后reloadJS运行一下，嘿，没问题了，能收发消息。。。