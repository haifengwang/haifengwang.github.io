---
layout: post
title: 用 WebSocket 实现服务端推送
category: HTML5
description: WebSocket 是 HTML5 一个重要特性，提供单 TCP 连接上进行全双工通讯的协议。本文用 WebSocket 实现一个简单推送。
keywords:  HTML5,WebSocket,推送
--- 

## 前言
最近学习 `SignalR` ，小总结 
[《通过  实现耗时任务实时通知》](http://haifengwang.github.io/blog/net/2015/07/15/a-net-SignalR_Real-time-Notifications.html) 。`SignalR` 本身融合利用了 Web 通信的好几种方式，其中之一用到了 **WebSocket** 。

## 简述
**WebSocket** 是 `HTML5` 提供的一种在单个 `TCP` 连接上进行全双工通讯的协议。`WebSocket API` 在 `JavaScript` 中的可用域（scope）包括 `DOM Window` 对象，和任何实现了 [WorkerUtils](https://developer.mozilla.org/zh-CN/docs/Web/API/WorkerUtils) 的对象。也就是说可以在 ` Workers` 中使用这些 API。

`WebSocket` 简单易用，可以很方便的使用。

###创建 `WebSocket` 对象

```
var Socket = new WebSocket(url, [protocal] );
```

###`WebSocket` 包含以下属性

+ Socket.readyState 只读属性，包含（0~3）四中状态。
+ Socket.bufferedAmount 只读属性，用来标识通过 `send()` 已经发送过的字节数。

###WebSocket Events

+ `open` 当 socket 建立连接时触发
+ `message` 接受到服务端消息时触发
+ `error` 发生错误时触发
+ `close` 连接断开时触发

### WebSocket 方法
+ `Socket.send()` 使用连接发送数据
+ `Socket.close()` 用于终止现有的连接

## 示例
通过一个简单 Demo 实现客户端浏览器通过 `WebSocket` 连接服务端后，服务的推送2条消息给客户端。

###客户端 HTML 和 Javascript (index.html)

```
<!DOCTYPE html>
<html lang="en">
    
    <head>
        <meta charset="UTF-8">
        <title>WebSocket Demo</title>
        <style>
        /**写这个例子时，一个同事写渐变效果，顺便写了这么一句*/
            .grad {
                  background: linear-gradient(cadetblue, blue); 
                  height: 200px;
                  width: 100%;
                }
        </style>
    </head>
    
    <body>
        <div id="content" class="grad">
            <div>WebSocket Demo</div>
             <div>
                  <a href="javascript:webSocketTest()">Run WebSocket</a>
             </div>
        </div>
        <script type="text/javascript">
        function webSocketTest(){
            var content = document.getElementById('content');
                if ("WebSocket" in window){
                   var socket = new WebSocket('ws://localhost:1337');
                    socket.onopen = function () {
                        socket.send('亲爱的服务器！我连上你啦！');
                    };
                
                    socket.onmessage = function (message) {
                        content.innerHTML += message.data +'<br />';
                    };
                
                    socket.onerror = function (error) {
                        console.log('WebSocket error: ' + error);
                    }; 
            }else{
               alert("您的浏览器不支持 webSocket");
            }
        }
            
        </script>
    </body>

</html>
```

### 服务端
服务端使用了 `Nodejs` 的 `websocket` 组件。

### 第一步通过 npm 安装
```
npm install websocket
```

###第二步创建websocket逻辑文件 （app.js）

```
var server = require('websocket').server, http = require('http');

var socket = new server({
    httpServer: http.createServer().listen(1337)
});

socket.on('request', function (request) {
	var connection = request.accept(null, request.origin);

	connection.on('message', function (message) {
		console.log(message.utf8Data);
		connection.sendUTF('欢迎你朋友！');
		setTimeout(function () {
			connection.sendUTF('朋友，既然来就多看看！');
		}, 1000);
		
	});

	connection.on('close', function (connection) {
		//客户端关闭和刷新时，都触发 close 事件
		console.log('connection closed');
	});
}); 

```

###第三步启动 websocket 服务
```
webSocketDemo  node app.js
```
### 运行 index.html 
在 `index.html` 页面就收到2条信息

```
欢迎你朋友！
朋友，既然来就多看看！
```

##参考资料
[https://developer.mozilla.org/zh-CN/docs/WebSockets/Writing_WebSocket_client_applications](https://developer.mozilla.org/zh-CN/docs/WebSockets/Writing_WebSocket_client_applications)

[http://www.tutorialspoint.com/html5/html5_websocket.htm](http://www.tutorialspoint.com/html5/html5_websocket.htm)

[https://github.com/theturtle32/WebSocket-Node](https://github.com/theturtle32/WebSocket-Node)
