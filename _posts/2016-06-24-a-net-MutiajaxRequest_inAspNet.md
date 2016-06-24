---
layout: post
title: ASP.NET 中 Ajax 并行请求遇到的问题
category: net
description: 最近在一个页面中用 Ajax 并行请求，本来看似简单的异步方式，在页面调转到详情页是发生了阻塞。对这个问题探索诊断，最会发现是 ASP.NET 的 Session 造成。
keywords: AJAX,ASP.NET,Session
---

## 背景

在一个页面中需要获取几家保险公司的报价数据，请求内容除了保险公司的标识外基本相似。有10家保险公司，就需要10个请求，这个10个请求在 `Controller` 对应一个 `Action`。我期待的结果，任何一个请求返回数据，就可以立即进入该结果`详情页面`（*详情内容在这次请求中已经获取*）。事实上，我点击以后，需要等待好久才可以进入详情页面。

页面是这样的：
![页面]({{ site:url }}/blog/images/2016/quote.png)

Ajax 代码大体是这样的:

```
for (var i = 0; i < 10; i++) {
    getQuote(i); //ajax请求
}
```
`详情页面`除了做微信 SDK 注册也没有其他复杂的逻辑。如果单独请求，速度还可以。但我从 `Ajax` 返回结果后，添加链接进入，慢的很不正常。

## 跟踪

面对这种情况，首先想到的是`详情页面`响应问题。先让详情页面变的足够简单。仍然解决不了问题。尝试在跳转处加上记录。

```
console.log("进入："+new Date().getTime());
location.href='@Url.Action("PriceDetails")'+'?source='+_sid;
console.log("结束："+new Date().getTime());
```
结束标志很快出现，确定还是`详情页面`响应慢导致，但没有想到慢的原因。跟踪网络，结果发现比较夸张的一幕。

![TTFB]({{ site:url }}/blog/images/2016/ttfb.png)

太夸张了，不过有一部分是本地网络原因。

** TTFB **
> **TTFB** (Time To First Byte)，是最初的网络请求被发起到从服务器接收到第一个字节这段时间，它包含了 TCP 连接时间，发送HTTP 请求时间和获得响应消息第一个字节的时间。

>优化方式通常是：
>+ 减少DNS查询
>+ 减少DNS查询
>+ 提早Flush
>+ 添加周期头

这种优化方式对这个页面而言不需要。但可以确定还是服务端响应比较慢造成。google TTFB 时间过长原因，看到这篇文章[Diagnosing Slow Web Servers with Time to First Byte](http://www.websiteoptimization.com/speed/tweak/time-to-first-byte/).其中列举了造成慢的一些因素。我比较关注这两个 **Too many processes / connections** 、**External resource delays** 。然后 google ASP.NET 阻塞。看到了这篇文章[ASP.NET Concurrent Ajax Requests and Session State Blocking](http://johnculviner.com/asp-net-concurrent-ajax-requests-and-session-state-blocking/)。作者遇到和我类似问题，他的解决方案，我拿来即用。

## 原理

**ASP.NET Session** 在所有 ASP.NET 应用域开启。 ASP.NET 的每个请求保持一个会话状态。这意味着如果两个不同的用户进行并发请求，每个请求占用一个会话信息。如果两个并发请求通过使用相同的 `SessionID` 值请求，第一个请求获取会话信息的独占访问。第二个请求需要等到第一个请求完成后才能执行。

怎么解决呢？

通过设置 **EnableSessionState** 状态只读，只读的会话信息不对导致对其他会话数据产生排他锁。在 ASP.NET MVC这样设置

```
[SessionState(System.Web.SessionState.SessionStateBehavior.ReadOnly)]
``` 
ASP.NET WebForm 需要在 `@Page` 指令下设置。

## 总结

并行请求在客户端利用 Ajax 发起看似很简单，但是如果服务端没有处理好，仍然达不到好的效果。在分布式的架构中有更多方式，对于我遇到的问题，简单一句就解决困扰我几天的问题。

