---
layout: post
title: Microsoft.NET Dynamic 类型不完全讲解
category: net
description: Microsoft .NET FrameWork4.0 新增了 dynamic 关键字。看似简单的一步，让C# 有了动态语言的特性。在处理外部接口，和其他语言交互时增加了极大的便利。笔者在最近频繁的处理 API接口的 JSON 数据时，用 dynamic 尝到了不少甜头，在此做一个总结。
keywords: Microsoft.NET,Dynamic,动态
---

Microsoft .NET FrameWork4.0 新增了 `dynamic` 关键字。看似简单的一步，让`C#` 有了动态语言的特性。在处理外部接口，和其他语言交互时增加了极大的便利。笔者在最近频繁的处理 API 接口的 JSON 数据时，用 `dynamic`,尝到了不少甜头，在此做一个总结。


## 简单的类型

```
dynamic da;
da = 1;
Console.WriteLine("值:{0},类型：{1}", da, da.GetType()); //[1]步奏
da = "2"; //[2]步奏2
Console.WriteLine("值:{0},类型：{1}", da, da.GetType());

```

输出结果：

```
值:1,类型：System.Int32

值:2,类型：System.String

```
去掉 [1] 步奏，会输出 `值:2,类型：System.String` ,`da`值和类型在 [2] 修改后都发生变化。

## 复杂类型

给 `dynamic` 赋值一个对象

```

    dynamic da;

    da = new { name="alex",age="18"};

    Console.WriteLine("值:{0},类型：{1}", da.name, da.GetType());

```

输出结果：

```
值:alex,类型：<>f__AnonymousType0`2[System.String,System.String]

```
注意观察，类型变化为

```
<>f__AnonymousType0`2[System.String,System.String] 

```

下面对类型做修改，添加属性。

```
da.sex = "男";

```

运行出现异常：

```
“Microsoft.CSharp.RuntimeBinder.RuntimeBinderException”类型的未经处理的异常在 System.Core.dll 中发生 

其他信息: “<>f__AnonymousType0<string,string>”未包含“sex”的定义

```
将上面写法做修改,先申明一个类

```
public class Person
    {
        public string Name { get; set; }

        public int Age { get; set; }

        public override string ToString()
        {
            return string.Format("Name={0} Age={1}",this.Name,this.Age);
        }
    }

```

然后：

```
dynamic da = new Person(); ;
da.Name="alex";
da.Age = 18;     
Console.WriteLine("值:{0},类型：{1}", da.ToString(), da.GetType());
```
运行结果：

```
值:Name=alex Age=18,类型：LazyExplore.Person  //LazyExplore是我的命名空间名称

```
动态添加 属性 `Sex`

```
 da.Sex = 1;
```

仍然会出现异常：

```
“Microsoft.CSharp.RuntimeBinder.RuntimeBinderException”类型的未经处理的异常在 System.Core.dll 中发生 

其他信息: “LazyExplore.Person”未包含“Sex”的定义
```
这一点可以说明 `dynamic` 类型在赋值后，对象的结构已经确定。


**再做一个尝试** 使用 `Dictionary`

```
Dictionary<string, object> dic = new Dictionary<string, object>();
dynamic da=dic;
da["name"] = "alex";
da["sex"] = "1";

Console.WriteLine("值:{0},类型：{1}", da["name"], da.GetType());
da["sex"] = 1;
Console.WriteLine("值:{0},类型：{1}", da["sex"], da.GetType());
```

输出结果：

```
值:alex,类型：System.Collections.Generic.Dictionary`2[System.String,System.Objec
t]
值:1,类型：System.Collections.Generic.Dictionary`2[System.String,System.Object]
```

看到这段，你是否想起了点什么？不防长话简说。

在.NET Framework 3.5 中增加程序集 `System.Web.Script.Serialization`。 有个类 `JavaScriptSerializer`,其中方法 `DeserializeObject`,将 json 字符串转换为 `C#` 对象，方法的签名是 `object` 类型，但在实际应用中一般是这样写。

```
private static readonly JavaScriptSerializer Serializer = new JavaScriptSerializer();
 
dynamic data = Serializer.DeserializeObject(result);
if (data.ContainsKey("Name"))
 {
     //根据 json 结构就可以这样很方便的解析 json 数据了
  }
 // data 的类型
  Console.WriteLine(data.GetType());
  //输出
{System.Collections.Generic.Dictionary`2[System.String,System.Object]}

```

这种方式对处理 `json` 数据增加了很大的灵活性，很方便的获取值或对值修改。

曾经要在 `C#` 中处理 `json` 数据，需要创建和 json 数据对应的实体的类，解析的时将 json 串反序列化为 `C#` 对象。

> *曾经写过一个通过 json 数据生成 C# 对象类的[项目](https://github.com/haifengwang/Json-ToCsharpObject)*

关于**性能的问题**，不在此话题之内。在计算机领域，没有银弹，任何好的方式都是基于场景权衡而来。

## 方法

+ `dynamic` 作为返回值的方法，和普通的没有多少区别。
+ `dynamic` 作为参数的方法。

```
 public static Person SayHello(dynamic d)
    {
        return new Person();
    }
```

调用时：

```
 dynamic da; //此时的 da 必须赋值
 var ps = SayHello(da); 
 //尽管明确表明了返回值类型，但是 ps 在未执行时为 dynamic 类型

```
**总结** `dynamic` 作为方法参数，在使用时必须有初始值，该方法返回值在运行时确定类型。

+ `dynamic` 不能为其添加扩张方法。

## 总结

以上从 `dynamic` 类型推断，到使用场景做简单的探索，除此而外 `dynamic` 还可以用于反射。

使用 **dynamic** 大家都会对对性能问题担忧。你的担忧是没有问题的，但是在日常工作中，性能和效率往往是针对一个场景做出一个选择。再次用那句话 

>No Silver Bullet ——**没有银弹**

你的任何选择，都是在一个场景下，在效率和性能之间的一次权衡。如果你的代码仅仅是执行某一次任务，或者非常低频的处理，也许你根本不用多虑了。