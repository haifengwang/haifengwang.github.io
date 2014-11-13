---
layout: post
title: CSS3 Multiple Columns & Column Breaks
category: HTML5
description: 用 CSS3 可以像报纸一样对文本多列排版,通过 Multiple Columns 和 Column Breaks 保证了完美显示
keywords:  CSS3,column-count,column-break-inside
--- 

用 CSS3可以像报纸一样对文本多列排版。其中主要属性有： 

+ **column-count**  具体列数
+ **column-gap**   列与列之间的距离
+ **column-rule**  设置列于列宽之间的样式（包括颜色、分割线等）

如下示例，排版为3列，列之间间隔为40px,有4px粗的绯红色线分割。

```
.newspaper {
    -webkit-column-count: 3;  /* Chrome, Safari, Opera */
    -moz-column-count: 3;  /* Firefox */
    column-count: 3;
    -webkit-column-gap: 40px;   /* Chrome, Safari, Opera */
    -moz-column-gap: 40px;  /* Firefox */
    column-gap: 40px;
    -webkit-column-rule: 4px outset #ff00ff; /* Chrome, Safari, Opera */
    -moz-column-rule: 4px outset #ff00ff; /* Firefox */
    column-rule: 4px outset #ff00ff;
}
```

其中`column-rule`包含一些其他相关属性，具体查看[**CSS3 column-rule Property**](http://www.w3schools.com/cssref/css3_pr_column-rule.asp) 

用以上的方式可以实现分栏显示，但会遇到整段内容被笨拙的分割跨列。如[示例](/code/column1.html)第4段的情况，本来完整的内容本被强制的拆分到另一栏。

当然了 **W3C**组织也不会让这种事情发生，因而出现了`column-break` 参考地址[ CSS column break properties](http://www.w3.org/TR/css3-multicol/#column-breaks)

通过如下修改就可以完美的解决

```
ol li{
    -webkit-column-break-inside:avoid;
    -moz-column-break-inside:avoid;
    -o-column-break-inside:avoid;
    -ms-column-break-inside:avoid;
    column-break-inside:avoid;
}
```
目前各个浏览器对`column-break`的表现形式还不相同。在*Chrome*下，内容逐条分栏，多出的内容在最后一栏完全显示。而*Safari*则将多出的内容截断（也是逐条分栏）。*Firefox*则是第一栏显示完整后，在将后面内容分到其他栏。而*IE*支持还不是太好！