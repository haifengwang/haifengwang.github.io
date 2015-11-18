---
layout: post
title: 探索 javascript 模板
category: javascript
description: 随着 web 发展，前端应用变得越来越复杂,ajax的广泛应用，交互的多层次性，以及不同场景的数据绑定和呈现，用传统的方式难以解决，于是 MVC思想流行，出现数据和界面分离，模板的应用成为重要的一环。
keywords: javascript,Template
--- 

最近一直和某系统打交道，其中前端数据的处理，大多是 `ajax` 加 `javascript` 模板实现。这种方式于绑定数据给予了极大的灵活性。

##曾经

曾经在前端页面处理 javascript 的数据绑定直接拼字符串，是这样实现的：

```
var data = [1,2]; 
var pvInnerHtml="<ul>";
for(var i=0;i<data.length;i++){
    pvInnerHtml+="<li>"+data[i]+"</li>";
}
pvInnerHtml+="</ul>";
$(“#ul_list”).html(“”).html(pvInnerHtml);
```
久而久之，由于每次都得重写拼凑一遍，效率低下，于是将这种代码仿照 `C#` 中的 `Format()` 方法 为 `String` 扩展了一个 `format`方法
>因为我是一名 `C#` 开发者

```
String.prototype.format = function(){
　　  var pattern = /\{(\d+)\}/gm;
　　  var args = arguments;
　　  var format = this.replace(pattern, function(){
　　      return args[arguments[1]];
　　  });
　　  return format;
　　}
　　
```
将曾经的代码替换为

```
var data = [1,2]; 
　　var result = [];
　　var template = "<li>{0}<li>";
　　for(var i=0,len=list.length; i<len, i++){
　　    result.push(template.format(data[i]));
　　}
　　$(“#ul_list”).html(“”).html(result.join(“”));
```

这种代码相对之前是方便了许多，但应对复杂的 `html`,数据和 view 之间还是有很高的耦合性，导致修改很不方便。

##探索

为了应对数据和复杂 `html` 片段之间的数据绑定想到模板。就如同 `ASP.NET` 以及等 Web 框架而言，都有比较发达的模板机制。模板使绑定数据和呈现上体现出很好的可读性，例如 ASP.NET  的 `Razor` 模板。

```
<ul>
    @for (int i = 0; i < 5; i++) {
        <li>@i</li>
    }
</ul>
```
`ASP.NET` 利用 `.NET` 的动态编译，将模板编译成动态的类，并利用反射动态执行类中的代码。这个过程是很复杂。

在 `javascript` 中，这样修改模板。

```
<ul>
    <% for(int i = 0; i < 5; i++){ %>
        <li><%=i %></li>
    <% } %>
</ul>

```
这种写法其实回到了 `ASP.NET WebForm` 模式，这种写法相比 `Razor` 模板在解析上相对容易一点。

**思路**：是通过正则表达替换通配符的部分解析赋值。

正则表达式：

```
var re = /<%([^%>]+)?%>/g;
```

尝试正则表达式匹配

```
while(match = re.exec(template)) {
    console.log(match);
}

```

输出：

```
["<%name%>", "name", index: 21, input: "<p>Hello, my name is <%name%>. I'm <%age%> years old.</p>"]

["<%age%>", "age", index: 35, input: "<p>Hello, my name is <%name%>. I'm <%age%> years old.</p>"]

```
因而模板引擎这样写：

```
var TemplateEngine = function(tpl, data) {
    var re = /<%([^%>]+)?%>/g, match;
    while(match = re.exec(tpl)) {
        tpl = tpl.replace(match[0], data[match[1]])
    }
    return tpl;
}

```
应用：

```
var template = '<p>Hello, my name is <%name%>. I\'m <%age%> years old.</p>';

TemplateEngine(template,{name:"alex",age:23})

//结果："<p>Hello, my name is alex. I'm 23 years old.</p>"

```

这种做法针对 `<p></p>` 简单的数据绑定可以实现。但作为模板需要应对的情况还有：
+ 复杂的对象(数据源)
+ 分支判断(`if-else`、`switch`等判断)
+ 循环(`for`循环)
+ 等其他情况


不到 20 行的代码的 `javascript` 模板引擎是这样实现的（*ps:不是我写的*）：

```
var TemplateEngine = function(html, options) {
    var re = /<%([^%>]+)?%>/g, reExp = /(^( )?(if|for|else|switch|case|break|{|}))(.*)?/g, code = 'var r=[];\n', cursor = 0, match;
    var add = function(line, js) {
        js? (code += line.match(reExp) ? line + '\n' : 'r.push(' + line + ');\n') :
            (code += line != '' ? 'r.push("' + line.replace(/"/g, '\\"') + '");\n' : '');
        return add;
    }
    while(match = re.exec(html)) {
        add(html.slice(cursor, match.index))(match[1], true);
        cursor = match.index + match[0].length;
    }
    add(html.substr(cursor, html.length - cursor));
    code += 'return r.join("");';
    return new Function(code.replace(/[\r\t\n]/g, '')).apply(options);
}

```
这个精简的模板引擎是通过 **正则表达式**、动态执行 `javascript` 字符串，渲染而完成字符串替换，完成数据绑定和呈现。

**问题**：
1. 性能：模板引擎渲染的时候依赖 `Function` 构造器实现，`Function` 与 `eval`、`setTimeout`、`setInterval` 一样，提供了使用文本访问 `javascript` 解析引擎的方法，但这样执行 `javascript` 的性能非常低下。
2. 调试：由于是动态执行字符串，若遇到错误调试器无法捕获错误源，导致模板 `BUG` 调试变得异常痛苦。在没有进行容错的引擎中，局部模板若因为数据异常甚至可以导致整个应用崩溃，随着模板的数目增加，维护成本将剧增。

##模板

面对这些问题，我如何解决呢？我找到了 **artTemplate** 。

`artTemplate`出自于 [糖饼](https://github.com/aui),鼎鼎大名的 `artDialog`也出于他。

根据 腾讯 [CDC](http://cdc.tencent.com/) 测试，artTemplate 在 chrome 下渲染效率测试中分别是知名引擎 Mustache 与 micro tmpl 的 25 、 32 倍。

特点：
+ 预编译 
> artTemplate 的编译赋值过程是在渲染之前完成的，这种方式称之为“预编译”
+ 更快的字符串相加方式
> 这个是在平均浏览器上说的，作者在里特意做了优化。

使用比较简单，和主流的其他没有多少区别。github 地址 [https://github.com/aui/artTemplate](https://github.com/aui/artTemplate)

##参考资料
[20行完成一个 javascript 模板](http://krasimirtsonev.com/blog/article/Javascript-template-engine-in-just-20-line)

[String.prototype.replace](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/replace)