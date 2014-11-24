---
layout: post
title: Swift 中的 Optional 与 C# 可空类型
category: ios
description: Swift 可以说吸取好多语言之长，在它的数据类型和语法结构等方面都可以看到其他语言的影子。其中Optional类型和 C#中的可空类型如出一辙。
keywords: Optional,Swift,C#,可空类型
---  

**Swift**可以说吸取众语言之长，在它的数据类型和语法结构等方面都可以看到其他语言的影子。下面主要谈`Optional`类型。

`Optional`类型,是 swift 中为了保证强类型的类型安全特意预设的一个泛型。


显式定义

```
var intOption:Int?

intOption=10;

println("\(intOption)") //这样并不能获取到值 Optional(10)
println("\(intOption!)") //通过“感叹号”取到值

var integerOption:Optional<Int>  //和上面的方式等价
integerOption=12

```

隐式定义

```
var name:String!
name="中国"

println("\(name)")

var nation:ImplicitlyUnwrappedOptional<String>

nation="印度"
```
隐式定义和显式定义的区别在应用层面主要在获取值的方式上。隐式获取值相对方便一点。

`Optional`类型在 swift 好多函数和方法中，被大量使用。

例如：

```
var value="1233d"

var intValue=value.toInt()

if(intValue != nil){
    println("\(intValue!)")
}
else{
    println("不是正确的数字")
}

```

再温习以下**C#** 可空类型.

C#中也是两种定义方式。功能上和 Swift  `Optional`类型很像。

```
			//这两种方式唯一的区别是前面的是后面的简写
			int? intValue;
			Nullable<int> integerValue;
			
			intValue=12;
			integerValue=12;
			int value=12;
			if(intValue==integerValue){
				Console.WriteLine("True");
			}
			
			if(intValue==value){
				Console.WriteLine("True");
			}
			else{
				Console.WriteLine("False");
			}
			
			if(integerValue.HasValue){
				
			}
```