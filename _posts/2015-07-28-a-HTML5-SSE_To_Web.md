---
layout: post
title: 服务器端消息推送之 Server-Sent Events 篇
category: html5
description: HTML5 推出 Server-Sent Events 允许客户端从服务器端获取更新。文本结合实例对如何创建、数据传递、自定义事件方面做探讨。
keywords:  HTML5,SSE,推送
--- 

## 概述
Server-sent Events 简称 **SSE**。服务器端通过 HTTP 或者专用的特定协议推送数据到 Web 页面。

### 创建
用 `EventSource` 的创建对象，并注册一个事件侦听器。

```
var source = new EventSource('updates.cgi');
source.onmessage = function (event) {
  alert(event.data);
};
```

### 事件流格式
服务器端发送的响应内容使用值为"text/event-stream"的MIME类型。

事件流仅仅是一个简单的文本,使用 `UTF-8` 格式的编码。每条消息后面都由一个空行作为分隔符，以冒号开头的行为注释行会被忽略。

>事件流总是解码为 `UTF - 8`,没有办法来指定另一个字符编码。

```
data: This is the first message.

data: This is the second message, it
data: has two lines.
```

这种使用默认事件类型 `message` 。也可以指定事件类型，将不同的事件区分。

```
event: add  //事件类型
data: 73857293

event: remove
data: 2153

event: add
data: 113411

```
在客户端这样监听自定义的事件

```
var source = new EventSource('updates.cgi');
source.addEventListener('add', addHandler, false);
source.addEventListener('remove', removeHandler, false);
```


事件流的请求可以使用 `HTTP 301` 和 307 重定向与正常的 HTTP 请求一样。如果连接被关闭，客户端将重新连接;客户端可以被告知使用 `HTTP 204` 无内容响应代码停止重新连接。

可以在客户端调用 `source.close();` 关闭链接。

使用 `SSE` ，比使用 `XMLHttpRequest` 或者 `iframe` 等用户代理方式可以更好地利用网络资源，实现客户端和服务器端协调。除此之外，还可以在移动端节省电量。



## API 重要点

+ **new EventSource(url)** 创建`EventSource`,并开始监听这个 url
+ **readyState** `EventSource`和 `XHR` 一样拥有此属性，标识连接状态。**0** 为正在连接、**1**连接打开、**2**为关闭连接。

+ **onopen, onmessage** `EventSource`对象监听的两个重要的事件，除服务器端自定义事件类型外，默认情况下，当有新消息被接受时都将触发`onmessage`。

+ **event.data** `Event`对象的属性，用于字符串消息传递。如果需要传递对象需要利用 `JSON` 编解码。
+ **close** 用户从客户端关闭连接。

+ **id** 事件ID,会成为当前 `EventSource` 对象的内部属性"最后一个事件ID"的属性值.

+ **retry**
一个整数值,表示浏览器在连接断开之后进行再次连接之前的等待时间(单位为毫秒,默认大约为3秒),如果该字段值不是整数,则会被忽略。

```
retry: 10000\n  //(将重新连接事件修改为10秒)
data: hello world\n\n
```

## 安全性

根据 WHATWG 的说明，在使用 SSE 接收消息时，需要检查消息的 e.origin 是否跟限定的相符合。

```
source.addEventListener('message', function(e) {
  if (e.origin != 'http://example.com') {
    alert('Origin was not http://example.com');
    return;
  }
  // ...
}, false);
```
这样避免被恶意利用。

## 示例

该示例试用 `nodejs` 作为服务端，客户端开启 `EventSource` 后，服务端推送登录成功，每隔5秒发送一次时间。

### 服务器端代码

```

var http = require('http');
var sys = require('sys');
var fs = require('fs');

http.createServer(function(req, res) {
  //debugHeaders(req);

  if (req.headers.accept && req.headers.accept == 'text/event-stream') {
    if (req.url == '/events') {
      sendSSE(req, res);
    } else {
      res.writeHead(404);
      res.end();
    }
  } else {
    res.writeHead(200, {'Content-Type': 'text/html'});
    res.write(fs.readFileSync(__dirname + '/sse-node.html'));
    res.end();
  }
}).listen(8000);

function sendSSE(req, res) {
  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive'
  });

  var id = (new Date()).toLocaleTimeString();

  // Sends a SSE every 5 seconds on a single connection.
  setInterval(function() {
    constructSSE(res, id, (new Date()).toLocaleTimeString());
  }, 5000);
  constructEventSSE(res,2014,"userlogon","{\"username\": \"John123\"}\n\n");
  constructSSE(res, id, (new Date()).toLocaleTimeString());
}

function constructSSE(res, id, data) {
  res.write('id: ' + id + '\n');
  res.write("data: " + data + '\n\n');
}

//自定事件类型
function constructEventSSE(res,id,ev,data) {
  res.write("event:"+ev+"\n");
  res.write("id:"+id+"\n");
  res.write("data:"+data+"\n\n");
}

function debugHeaders(req) {
  sys.puts('URL: ' + req.url);
  for (var key in req.headers) {
    sys.puts(key + ': ' + req.headers[key]);
  }
  sys.puts('\n\n');
}
```

### 客户端

```
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8" />
    <title>SSE demo</title>
     <script>
    function getData() {
        if (!!window.EventSource) {
            var source = new EventSource('/events');
            source.addEventListener('open', function(e) {
                console.log("建立连接成功！");
            }, false);

            source.addEventListener('error', function(e) {
              if (e.readyState == EventSource.CLOSED) {
                console.log("关闭！");
                
              }
              //e.readySate==undefined 将服务关闭，否则会一致请求
              if(!e.readyState){
                   source.close();
              }
           }, false);
           
           //登录成功
           source.addEventListener('userlogon', function(e) {
              var data = JSON.parse(e.data);
              console.log('User login:' + data.username);
            }, false);
            
            //一般性的事件，服务器端不需要写事件名
           source.addEventListener("message",function(e){
                document.getElementById("message").innerHTML += '<p>' + e.data + '</p>';
           },false);
           // source.onmessage = function(e) {
               
            // };

        } else {
            alert("浏览器不支持SSE");
        }

    }
    </script>
</head>

<body>
    <h1>SSE 示例</h1>
    <div>
        <p>获取服务器端数据</p>
        <input type="button" value="获取数据" onclick="getData();" />
        <div id="message"></div>
    </div>
   
</body>

</html>

```
>注：代码来源于 HTML5Rocks

## 参考资料
[http://www.ibm.com/developerworks/cn/web/1307_chengfu_serversentevent/](http://www.ibm.com/developerworks/cn/web/1307_chengfu_serversentevent/)

[https://developer.mozilla.org/zh-CN/docs/Server-sent_events/Using_server-sent_events](https://developer.mozilla.org/zh-CN/docs/Server-sent_events/Using_server-sent_events)

[http://www.html5rocks.com/en/tutorials/eventsource/basics/?redirect_from_locale=zh](http://www.html5rocks.com/en/tutorials/eventsource/basics/?redirect_from_locale=zh)

[W3C](http://www.w3.org/TR/eventsource/#the-eventsource-interface)
