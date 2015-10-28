---
layout: post
title: ios App集成支付宝遇到一些问题
category: ios
description: 项目中用到了支付宝，集成时出现了不少问题。其中有对机制理解不够的问题，也有支付宝本身有些搞不清楚的逻辑。
keywords: ios,支付宝,集成
--- 

##错误1：openssl rsa.h file not found

集成支付宝，首先在支付宝账户下下载最新的SDK,将文件拷贝到app目录下，然后出现 `openssl rsa.h file not found`

通过查找资料：

需要openssl这个文件导入到了这个工程目录下，然后在`Header Search Paths `设置`$(SRCROOT)/项目名/Alipay`，问题就解决了

[参考地址](http://blog.csdn.net/pearlhuzhu/article/details/9300435)


##错误2：“系统繁忙，请稍后再试”无法支付

按照以上方法集成后，开始测试支付，结果出现这样的错误
>
Error Domain=系统繁忙，请稍后再试 Code=1000 \"The operation couldn’t be completed. (系统繁忙，请稍后再试 error 1000.)\

在网上查了好多资料，大多说法是 密钥不匹配。

解决方式：

1. 检查密钥，然后在重设置公玥。（这个步骤走了好几篇。在一片博客上说生成必须在本机。之前在windows上利用支付宝提供的工具，又改为2. c上。*和支付宝小二确认，是不是本机都行*）


2. 密钥设置好后，还有问题，最后求助支付宝小二，原来是请求参数出现错误。

[设置公玥](https://cshall.alipay.com/enterprise/help_detail.htm?help_id=473890)

MAC OS自openSSL

`localhost:~ root$openssl` 

然后一步步的生成密钥

##错误3：在安装支付宝的客户端的终端上获取不到付款状态

app调用支付宝分两种情况：

1.客户端没有安装支付宝。这时支付宝SDK用一个`webview`实现web版的调用。这种情况下按照文档方法，一切正常。

2.客户端安装了支付宝情况下。

demo中例子是这样的

``` 
  if ([url.host isEqualToString:@"safepay"]) {
      
         [[AlipaySDK defaultService] processAuth_V2Result:url
                                       standbyCallback:^(NSDictionary *resultDic) {
          NSLog(@"result = %@",resultDic);
          NSString *resultStr = resultDic[@"result"];
        }];

  }
  
```
调试后，这个方法根本不执行！


**集成文档中是这样提示的：**

>设备已安装支付宝客户端情况下,处理支付宝客户端返回的 url。
`-(void)processOrderWithPaymentResult:(NSURL*)resultUrl standbyCallback:(CompletionBlock)completionBlock;`NSURL *resultUrl
支付宝客户端回传的 url
CompletionBlock completionBlock
当支付宝客户端在操作时,商户 app 进程在后台被结束,只能通过这个 block 输出支付 结果。

尝试后，这个执行没有问题。还好正确的方法也是支付宝提供的。但是示例中的那个到底是干什么的呢？算了，问题算解决了！