---
layout: post
title: 提交 App 到 App Store遇到的问题
category: ios
description: 今天是 Xcode 更新后，以及苹果 ios8 以后提交第一个应用。其中遇到了不少问题。第一个证书问题没法提交。第二个就是各种图片问题。
keywords: 开发,ios,问题,P12,AppStore
---  

今天是 Xcode 更新后，以及苹果 ios8 以后提交第一个应用。其中遇到了不少问题。第一个证书问题没法提交。（第一次用这个开发账号）。

## P12 文件 

注册正式成为 apple 开发者（花费￥99/年），得到Certification 证书（.cer文件）。 证书是对电脑开发资格的认证，每个开发者帐号有一套，分为两种： 

-  **Developer Certification**(开发证书) 安装在电脑上提供权限：开发人员通过设备进行真机测试。可以生成副本供多台电脑安装（只需要在该账户的开发中心Certificates, Identifiers & Profiles下载就行）；
- **Distribution Certification**(发布证书) 安装在电脑上提供发布iOS程序的权限：开发人员可以制做测试版和发布版的程序。

安装证书成功后会生成**Keychain**，这个证书副本通过配置证书的电脑导出Keychain（.p12文件）安装到其他机子上，让其他机子得到证书对应的权限。

发布证书（Distribution Certificate）是一个后缀为 .p12 的文件（Certificates.p12）； 在Mac 系统下， 双击这个文件，这个证书会自动导入到 Mac 下的 key chain （钥匙链） 目录下。（如果证书提供者做了加密，打开的时间需要输入加密密码）。

[参考资料](http://blog.csdn.net/gtncwy/article/details/8617788)

---
由于第一次使用这个账号，提交的时间才想到并没有给我 P12文件，这个账号还是离职的同事移交下来的，费了一定的周折才搞到 P12文件。

##App 各种图片
各种尺寸的图标，App 图和启动图缺一不可。之前Default-568h@2x.png只要目录下存在就行，在新版本的 xcode项目设置中，比如对应到相应的机型上。

其中 App图标57*57的，在Images.xcassets 由于屏幕下没有看到，导致茫然了好一会儿。最后缩小右边的侧栏才发现。

**Screenshots 图片**之前有些尺寸可以不添加，现在所有尺寸必须全部提交。

[参考资料](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/IconMatrix.html)


