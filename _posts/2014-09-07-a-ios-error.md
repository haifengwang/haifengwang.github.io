---
layout: post
title: 最近ios开发中遇到的几个问题总结
category: book
description: 在ios开发中，最近由于开发环境，已经程序环境配置发生一些错误。这些错误会直接导致程序无法运行，但并不是程序错误，总结入邪
keywords: 开发,ios,error
---  


**第一种:缓存文件问题**

“Could not launch 'app name'”，No such file or directory (/Users/appleapple/Library/Developer/Xcode/DerivedData/FCHK-bdgaoltxzddyrogqdfuhtasreoxs/Build/Products/Debug-iphoneos/FCHK.app)

这个错误不是程序问题，而是`xcode`缓存文件造成。

解决办法：

退出`xcode` ,然后进入终端将：`MacBook-Pro:/ user$ rm -rf /Users/username/Library/Developer/Xcode/DerivedData/*`

然后重启就搞定了。

**第二：类库添加或引入问题**

在xcode 开发过程中，经常遇到如下错误:
 "_OBJC_CLASS_$_ClassName", referenced from:

我总结了两种错误情况： 

+ 一类所在的库（框架）没有被引入，常见为framework添加。 

>解决方法：添加该类所在的库（框架）

+ 一类没有被正确的添加到工程中，这种情况在工程中的确存在这个类，可是程序就是不能识别。
>解决方法：选中该类，只删除它在该工程中的引用，然后，再添加引用，如"Undefined symbols for architecture i386: "OBJC_CLASS_$_RiskViewController" referenced from" 
错误时，有可能某文件没有引进，可以打开项目文件夹，看看文件是不是映入状态。

**第三：提交发布问题**

在XCODE中提交app时出现“no application records were found”错误提示  

出现这种错误，是因为在iTunes Connect中少操作了一个步骤，app的状态还是“Ready for upload”，点击view Details，右下角或右上角有个Ready to Upload Binary，点击后app的状态变成waiting for upload。接下来就可以返回到xcode中提交app啦。

**第四：嵌入友盟反馈页面问题**

程序嵌Umeng的反馈页面，但是输入框不显示，只要在info.list中将`View controller-based status bar appearance` 设置为`NO`，就可以。切记不要再程序中添加 `[[UIApplication sharedApplication] setStatusBarHidden:YES withAnimation:UIStatusBarAnimationFade]`;
这样操作保证以全屏的方式弹出反馈页面（可以显示反馈框，否则不能显示）。
