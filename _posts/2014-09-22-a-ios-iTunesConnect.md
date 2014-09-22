---
layout: post
title: 新版 iTunesConnect 的使用的中出现的错误和坑
category: ios
description: 随着 IOS8 发布,developer.apple.com 也做了改版。改版后的 iTunesConnect 和之前的使用方式大相径庭，为应用的分发造成了很大不便。
keywords: 开发,ios,error
--- 

最近提交了一个应用，本来只是一次的版本升级，仅仅做了一些优化，对实质性的内容的没有多少修改，但是仍然被打回。也许上次通过比较幸运，也许那天审核人员的心情比较好。没有办法只有认灾。又做了几处简单修改，继续发布。

然后出现了这样的错误
>ERROR ITMS-9000: "Redundant Binary Upload. There already exists a binary upload with build version 

按照常理想，应该在 iTunesConnect 中将原来的删除，结果找了很久没有办法删除。新版的界面修改的操作很不方便。

好多网站的改版都讲究平滑的过渡，但是苹果不讲究这个，因为是他们是苹果，我们只有适应他们的变化。不过这个问题终于找到了解决办法。

**app里面有一个version和build，build不同就可以上传多个同一version的文件
修改了一下build的值，重新上传就可以了。** [参考链接](http://stackoverflow.com/questions/25680604/error-itms-9000-redundant-binary-upload-there-already-exists-a-binary-upload)

这样修改以后提交成功，但这是仅仅提交成功,App状态还是**Prepare for Submission**，还需要手动**Submit for Review**，几个步骤执行下来，终于可以为**Waiting For Review**这个时间就等待最后的宣判。

由于新版增加了测试功能，所以上传并不自动提交。目前还不清楚上传后怎么测试。最大的感觉和老版相比使用很不方便。但这个吐糟是没有用的，只有适应。因为他们是苹果，当其他人说“Yes”，苹果说“No”，要用它的你只有跟随他说“NO”.