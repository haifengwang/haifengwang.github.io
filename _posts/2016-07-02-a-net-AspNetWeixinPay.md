---
layout: post
title: 在ASP.NET中利用JSAPI实现微信支付
category: net
description: 用 JSAPI 实现微信支付，本应按照官方文档一步步就可以搞定，做了才知道微信给大家增加好多难度。如果你探索不到位，实现起来并不是一件容易的事情。笔者在做这一块时就踩过好多坑，一一填满后，流出了两行心酸的泪😢。
keywords: Pay,ASP.NET,微信
---

> 微信支付的坑比月球表面还要多。————某开发者说

## 前期工作

用微信 JSAPI 做支付，首先确保公众号(*或服务号*)开通支付功能。支付功能需要单独申请。

拥有支付权限了后，登录公众号，在左侧菜单中找到【**微信支付**】,设置**支付目录**和**测试目录**，以及**测试的白名单**。
如图：

![设置]({{ site:url }}/images/pay/paySetting.png)

进入【**使用教程**】-->**公众号支付**-->**支付步奏**，其中有一个菜单是 【**H5调起支付API**】,有一段调用 JS 代码：

![h5Pay]({{ site:url }}/images/pay/h5pay.png)

支付需要的具体参数在[统一下单](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_1)中。

在[微信公众平台开发者文档](http://mp.weixin.qq.com/wiki/7/aaa137b55fb2e0456bf8dd9148dd613f.html)也有微信JS-SDK说明文档。

其中下单 JS 代码是这样的：

![支付]({{ site:url }}/images/pay/pay.png)

这里的代码和支付文档中的不一致，经确定这是属于 V3 最新版本。集成有如下步骤：

![设置]({{ site:url }}/images/pay/buzou.png)

## 支付前

写支付代码时，确保能够正常的获取 `openid`,调用支付需要用。

获取 openid 需要以下几步，具体文档[微信公众平台开发wiki-->微信网页开发](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140842&token=&lang=zh_CN)。

1. 用户同意授权，获取 `code`

2. 通过 `code` 获取网页授权的 `access_token`

3. 拉取用户信息（`openid` 就在用户信息中）

[*注*]如果需要要刷新网页授权 `access_token`，避免过期。

这些返回参数都是`JSON`(*这部分和支付应该不是一个团队的产物*)。

## 支付

支付基本步骤：

1. 引用微信 JS (http://res.wx.qq.com/open/js/jweixin-1.0.0.js)

2. 设置 config 参数(包括appId、timeStamp、nonceStr、signature,除appId外都需要后端生成)

3. 设置 chooseWXPay 参数(需要使用的JS接口列表，支付用 `chooseWXPay`)

4. 支付(根据[统一下单文档](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=4_3)组装 xml 数据，`post`到`https://api.mch.weixin.qq.com/pay/unifiedorder`,获得`prepay_id`,prepay_id 用于数字签名，然后通过`JSAPI`就可以支付)

传说中的坑来了。js中的 `nonceStr` `package`  `paySign` 需要后端生成，具体代理如下：

1. 加密中串键值前后端有区别，命名规则发生变化，区分大小写。

2. 必须遵守一些规则(把所有的参数首字母从小到大传参的形式组成字符串后，把key值再拼接上),具体在规则在[安全规范](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=4_3)和[统一下单]的文档中。

服务器端代码(*我使用了 Senparc SDK*)用于获取 `prepay_id`,并生成签名。

```
                string timeStamp = "";
                //随机串
                /**
                * 微信文档中是这样写的。
                * 微信支付API接口协议中包含字段nonce_str，主要保证签名不可预测。我们推荐生成随机数算法如下：调用随机数函数生成，将得到的值转换为字符串。
                */
                string nonceStr = "";
                string paySign = "";
                //创建支付应答对象
                RequestHandler packageReqHandler = new RequestHandler(null);
                //初始化
                packageReqHandler.Init();

                timeStamp = TenPayV3Util.GetTimestamp();
                nonceStr = TenPayV3Util.GetNoncestr(); //这里用封装好的。
                //设置package订单参数
                packageReqHandler.SetParameter("appid", TenPayV3Info.AppId); //公众账号ID
                packageReqHandler.SetParameter("mch_id", MchId); //商户号
                packageReqHandler.SetParameter("nonce_str", nonceStr); //随机字符串
                packageReqHandler.SetParameter("body", "短信充值");
                packageReqHandler.SetParameter("out_trade_no", orderID); //商家订单号(根据业务自己创建)
                packageReqHandler.SetParameter("time_start", DateTime.Now.ToString("yyyyMMddHHmmss"));
                packageReqHandler.SetParameter("time_expire", DateTime.Now.AddMinutes(10).ToString("yyyyMMddHHmmss"));
                //商品金额,以分为单位
                packageReqHandler.SetParameter("total_fee", "1");
                packageReqHandler.SetParameter("spbill_create_ip", Request.UserHostAddress); //用户的公网ip，不是商户服务器IP
                packageReqHandler.SetParameter("notify_url", "url"); //接收财付通通知的URL
                packageReqHandler.SetParameter("trade_type", "JSAPI"); //交易类型，一定是JSAPI
                packageReqHandler.SetParameter("openid", "openId"); //用户的openId
                string sign = packageReqHandler.CreateMd5Sign("key", "appsecret ");
                packageReqHandler.SetParameter("sign", sign); //签名
                string data = packageReqHandler.ParseXML();
                //将这些数据发送给微信 
                /**
                *按照统一下单：https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_1#
                *发送的接口URL:https://api.mch.weixin.qq.com/pay/unifiedorder
                */
                var result = TenPayV3.Unifiedorder(data);
                    var res = XDocument.Parse(result);
                    string prepayId = string.Empty;
                    //获取预支付成功时
                    //保存预支付ID，防止用户后退重复下单
                    if (res.Element("xml").Element("result_code").Value == "SUCCESS")
                    {
                        prepayId = res.Element("xml").Element("prepay_id").Value;
                    }
                //预支付ID后面需要用
                /**关键的地方来*/
                 RequestHandler paySignReqHandler = new RequestHandler(null);
                    paySignReqHandler.SetParameter("appId", TenPayV3Info.AppId);
                    paySignReqHandler.SetParameter("timeStamp", timeStamp);
                    paySignReqHandler.SetParameter("nonceStr", nonceStr);
                    paySignReqHandler.SetParameter("package", string.Format("prepay_id={0}", prepayId));
                    paySignReqHandler.SetParameter("signType", "MD5");
                    paySign = paySignReqHandler.CreateMd5Sign("key", TenPayV3Info.Key);
                    //appId必须这样写
                    ViewData["appId"] = TenPayV3Info.AppId;
                    //timeStamp必须这样写
                    ViewData["timeStamp"] = timeStamp;
                    ViewData["nonceStr"] = nonceStr;
                    ViewData["package"] = string.Format("prepay_id={0}", prepayId);
                    ViewData["paySign"] = paySign;
                    ViewData["signature"] = 
                    new JSSDKHelper().GetSignature(base.JsapiTicket, nonceStr, timeStamp, Request.Url.ToString());
                    /*timeStamp、nonceStr 必须按照大小规则来，不然校验不通过*/
```
对应页面中 JS 代码：

```
 <script src="http://res.wx.qq.com/open/js/jweixin-1.0.0.js"></script>
    <script type="text/javascript">
        wx.config({
            debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
            appId: '@ViewData["appId"]', // 必填，公众号的唯一标识
            timestamp: "@ViewData["timeStamp"]", // 必填，生成签名的时间戳
            nonceStr: '@ViewData["nonceStr"]', // 必填，生成签名的随机串
            signature: '@ViewData["signature"]',// 必填，签名，见附录1
            jsApiList: ['chooseWXPay'] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
        });
        wx.ready(function () {
            document.querySelector('#chooseWXPay').onclick = function () {
                wx.chooseWXPay({
                    "appId": '@ViewData["appId"]',
                    "timestamp": '@ViewData["timeStamp"]', // 支付签名时间戳，注意微信jssdk中的所有使用timestamp字段均为小写。但最新版的支付后台生成签名使用的timeStamp字段名需大写其中的S字符
                    "nonceStr": '@ViewData["nonceStr"]', // 支付签名随机串，不长于 32 位
                    "package": '@Html.Raw(ViewData["package"])', // 统一支付接口返回的prepay_id参数值，提交格式如：prepay_id=***）
                    "signType": "MD5", // 签名方式，默认为'SHA1'，使用新版支付需传入'MD5'
                    "paySign": '@ViewData["paySign"]', // 支付签名
                    success: function (res) {
                        // 支付成功后的回调函数
                    },
                    fail: function (res) {
                        alert("错误：" + JSON.stringify(res));
                    }
                });
            };
        });
    </script>
```
至此调用支付代码完成。我遇到的问题：

+ 由于之前一个同事做过微信支付，直接用老版代码，结果不能弹出支付窗口。

+ `wx.config` 配置有问题，导致无法执行。

这些完成后，可以支付了。 

## 回调

根据文档[https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_7](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_7)在回调地址的处理逻辑中以 `xml`的形式发送数据到服务器端。

示例：

```
<xml>
  <return_code><![CDATA[SUCCESS]]></return_code>
  <return_msg><![CDATA[OK]]></return_msg>
</xml>

```
在回调地址的那个 `Action` 中，这样处理：

```
                RequestHandler reqHandler = new RequestHandler(null);
                ResponseHandler resHandler = new ResponseHandler(null);
                var orderWeiXinNotify = new OrderWeiXinNotify()
                {
                    result_code = resHandler.GetParameter("result_code"),
                    appid = resHandler.GetParameter("appid"),
                    mch_id = resHandler.GetParameter("mch_id"),
                    device_info = resHandler.GetParameter("device_info"),
                    err_code = resHandler.GetParameter("err_code"),
                    err_code_des = resHandler.GetParameter("err_code_des"),
                    openid = resHandler.GetParameter("openid"),
                    is_subscribe = resHandler.GetParameter("is_subscribe"),
                    trade_type = resHandler.GetParameter("trade_type"),
                    bank_type = resHandler.GetParameter("bank_type"),
                    transaction_id = resHandler.GetParameter("transaction_id"),
                    out_trade_no = resHandler.GetParameter("out_trade_no"),
                    time_end = resHandler.GetParameter("time_end")
                };
                resHandler.SetKey(TenPayV3Info.Key);
                if (resHandler.IsTenpaySign() && orderWeiXinNotify.result_code == "SUCCESS")
                {
                    reqHandler.SetParameter("return_code", "SUCCESS");
                    reqHandler.SetParameter("return_msg", "OK");
                    string data = reqHandler.ParseXML();
                    var res = TenPayV3.Unifiedorder(data);
                }
```
基本按照官方文档做，好多博客上也是这样写的。但是事实告诉我，这种做法是不对的。

然后按照之前同事写过的方式，将以上修改为：

```
                if (resHandler.IsTenpaySign() && orderWeiXinNotify.result_code == "SUCCESS")
                {
                    return Content("success");
                }
                return Content("falid");
```
回调成功了！

这样整个流程基本算走通了。没有完成的喜悦，只是长舒一口气，这个过程太『曲折』，尽然在一步步的猜测中做出来。

## 总结

微信官方文档混乱，版本不一致(公众号开发文档和JSSDK文档)，规则不统一(有些数据是`XML`格式，有些是`JSON`),参数描述过于简单。对接入的开发者造成了很大的困扰，似乎大多数开发者都是猜测对接。论坛上关于微信支付的帖子很多，评论中骂声一片。不知道腾讯作为一个巨头型的公司，江湖传说国内互联网巨头中产品做的最好的，何时能对开发者友好一点。（*当年接入支付宝，支付宝每一个接口都对应一个DEMO示例压缩包，压缩包中有文档，有 demo，当时觉得作为一个大公司没有统一的 SDK*）经过微信这一遭，似乎明白了，没有糟的，只有更糟的。