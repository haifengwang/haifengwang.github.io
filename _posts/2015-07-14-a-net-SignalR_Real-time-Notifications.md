---
layout: post
title: 通过 SignalR 实现耗时任务实时通知

category: net
description: 通过 SignalR 实现一个服务器端执行一个耗时任务，执行完通知客户端。
keywords: ASP.NET,SignalR,realtime
--- 


## 简述

**ASP.NET SignalR** 是一个为 ASP.NET 开发者提供简化  `real-time web` 应用的库。简化并优化以往 `Web` 程序耗资源 `javascript轮询`，以及 `Comet Programming` 方案，同时为 Web程序 添加了即时能力（realtime）。

`ASP.NET SignalR` 出现后就受到广泛的关注。目前已经到升级到2.0,需要 `.NET Framework 4.5`支持。


## 目标

利用 **ASP.NET SignalR** 实现一个服务器端执行一个耗时任务，执行完通知客户端。

## 实现

### 第一步：安装 ASP.NET SignalR
通过 `Nuget` 安装 `ASP.NET SignalR`

```
Install-Package Microsoft.AspNet.SignalR

```

安装内容包括 `ASP.NET SignalR`,`Microsoft.AspNet.SignalR.JS 2.0.0`，以及相关依赖。

### 第二步：注册 `SignalR `的中间件 

```

using System;
using System.Threading.Tasks;
using Microsoft.Owin;
using Owin;

[assembly: OwinStartup(typeof(RealtimeNotifier.Startup))]

namespace RealtimeNotifier
{
    public class Startup
    {
        public void Configuration(IAppBuilder app)
        {
            // For more information on how to configure your application, visit http://go.microsoft.com/fwlink/?LinkID=316888
            app.MapSignalR();
        }
    }
}

```

通过注册中间件，注册了客户端访问服务端 `Hub` 的路由。默认路由地址 `/signalr/hubs`,既客户端通过代理生成 javascript 文件的地址。

可以通过配置修改默认地址

```
app.MapSignalR("/signalr", new HubConfiguration());
```
这里修改后，客户端也要做相应的改变

```
$.connection.hub.url = "/signalr"
$.connection.hub.start().done(init);
var connection = $.hubConnection("/signalr", { useDefaultPath: false });

```

### 第三步：创建 `Hub` 类

```
using System.Threading;
using Microsoft.AspNet.SignalR;

namespace RealtimeNotifier
{
    public class RealtimeNotifierHub : Hub
    {
        public int recordsToBeProcessed = 100000;

        public void DoLongOperation(int num)
        {
            for (int record = 0; record <= recordsToBeProcessed; record++)
            {
                if (ShouldNotifyClient(record))
                {
                    if ( num==1) {
                        Clients.Caller.sendMessage(string.Format("成功!"));
                    }
                    if (num == 2) {

                        Clients.Caller.sendMessage(string.Format("发送--{0}!",record));
                    }
                    
                    Thread.Sleep(10);
                }
            }
        }

        private static bool ShouldNotifyClient(int record)
        {
            return record % 10 == 0;
        }
    }
}

```

在 `RealtimeNotifierHub` 中，`DoLongOperation` 模拟一个很耗时的方法，客户端可以通过调用这个方法，获取最后的执行结果。

该 `Hub` 在客户端生成代理

```
 var realtimeNotifier = $.connection.realtimeNotifierHub;
```
可以通过 `HubNameAttribute`修改客户端代理的名称

```
//Server

[HubName("NotifierHub")]
public class RealtimeNotifierHub : Hub

//JavaScript client 

var NotifierHubbProxy = $.connection.NotifierHub;

```

**注意点**如何选择客户端 `PRC` 通道

+ 面向所有客户端

```
Clients.All.sendMessage(msg);
```
+ 面向请求客户端

```
  Clients.Caller.sendMessage(string.Format("成功!"));
```
+ 面向指定客户端

```
Clients.Client(Context.ConnectionId).sendMessage(msg);

```

除此之外还有面向特定组，特定用户名等。

### 第四步：Web 客户端

```
<html>
<head>
    <title>Real-time notifier</title>
    <script src="Scripts/jquery-1.10.2.min.js"></script>
    <!-- Import the SignalR scripts -->
    <script src="Scripts/jquery.signalR-2.0.0.min.js"></script>
    <script src="/signalr/hubs"></script>
</head>
<body>
    <div style="margin-top: 100px;">
        <input type="text" id="txt" placeholder="输入数字" />
        <!-- This button will trigger the time consuming operation -->
        <input type="button" id="mybutton" value="Call a time consuming server side operation" />
    </div>
    <script type="text/javascript">

        $(function () {
            // 初始化服务端链接
            var realtimeNotifier = $.connection.realtimeNotifierHub;
            
            realtimeNotifier.client.sendMessage = function (message) {
                console.log(message);
                
            };

            // 建立链接
            $.connection.hub.start().done(function () {
                $('#mybutton').click(function () {
                    var va = $("#txt").val();
                    realtimeNotifier.server.doLongOperation(parseInt(va));
                });
            });
        });
    </script>
</body>
</html>

```

通过以上四步实现了，一个服务端耗时任务执行完后对客户端即时通知。

## 参考资料

[http://www.asp.net/signalr/overview/getting-started](http://www.asp.net/signalr/overview/getting-started)

[http://www.codeproject.com/Tips/672433/Real-time-Notifications-with-SignalR](http://www.codeproject.com/Tips/672433/Real-time-Notifications-with-SignalR)

感谢 [GustavoMartins](http://www.codeproject.com/script/Membership/View.aspx?mid=1076070) 文中例子是在他文章基础上的修改。