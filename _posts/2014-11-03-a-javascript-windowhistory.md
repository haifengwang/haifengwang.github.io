---
layout: post
title: 探究 Window.history
category: javascript
description:  History 对操作浏览器的会话历史提交接口。HTML5引进了 history.pushState()方法和 history.replaceState()方法，它们允许你逐条地添加和修改历史记录条目，丰富了Web的操作方式。
keywords: History,javascript,pushState
--- 

`window.history` **Mozilla MDN**这样描述:window.history 是一个只读属性，指向 `History`对象，对操作浏览器的会话历史提交接口。

在 Chrome控制台中查看**History** 

```
History {
	state: null,
    length: 1, 
    pushState: function, 
    replaceState: function,
     back: function…}
```
提供几个方法 `back()` `forward()` `go` 主要用来前进后退，所有浏览器都支持。

**属性**

`History.length`返回浏览器会话列表的的元素包含当前页。如果新 Tab 加载页面返回1.
>IE 6 和 Opera 9 以 0 开始，而 Firefox 1.5 以 1 开始

`History.current` 

`History.next` 

`History.previos`
>表示当前、下一个、前一个会话中 DomString 的 URl。个人感觉很有用，不过很遗憾，并不是 *W3C* 标准。仅仅 Gecko 26支持。

HTML5引进了`history.pushState()`方法和`history.replaceState()`方法，它们允许你逐条地添加和修改历史记录条目。这些方法可以协同`window.onpopstate`事件一起工作。

写了一个例子，通过`history.pushState()`修改链接，查看一下状态。结果发现在文件路径下(file:///Users/Documents/html/hash.html)下不执行,配置成站点才可以。

``` 
$("input").click(function(event) {
		console.log("触发");
		var json={time:new Date().getTime()};
		window.history.pushState(json,"修改","/test.html/new");
		var currentState = history.state;
		console.log(currentState);
	});
$(window).bind("popstate", function(e) {
    alert("location: " + document.location + ", state: " + JSON.stringify(e.state));
  });

```

页面上有一个`input`的按钮，触发点击事件。进行`window.history.pushState`后，并不能触发`popstate`。`popstate`只有在做出浏览器动作时，才会触发该事件，如用户点击浏览器的回退按钮（或者在Javascript代码中调用history.back()）。但`e.state`永远是 `undefined`。
>[Mozilla MDN](https://developer.mozilla.org/zh-CN/docs/Mozilla_event_reference/popstate) 的例子，不知道我错在哪里。

不同的浏览器在加载页面时处理`popstate`事件的形式存在差异。页面加载时Chrome和Safari通常会忽略`popstate`事件，但Firefox则会触发该事件。*（出自 Mozilla MDN 和验证吻合）*




