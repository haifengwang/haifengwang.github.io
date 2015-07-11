---
layout: post
title: C#中几种延迟执行(Deferred Execution)的探讨

category: net
description: 在 .NET 程序中好多地方都有延迟(Lazy)的痕迹,这一点在 Linq 中发扬广大。在 .NET4.0 中又引入了 System.Lazy<T> 更加丰富了延迟的方式。
keywords: C#,dotNet,延迟执行,Deferred Execution
--- 


在 `.NET` 程序中好多地方都有延迟(`Lazy`)的痕迹,这一点在 `Linq` 中发扬广大。在 `.NET4.0` 中又引入了 `System.Lazy<T>` 。更加丰富了延迟的方式。

延迟主要有：延迟实例化，延迟初始化，延迟执行等。主要表达的思想是，把对象将会延迟到使用时创建，而不是在对象实例化时创建对象，即用时才加载。

这种方式有助于提高于应用程序的性能，避免浪费计算，节省内存的使用等。

## Linq中延迟

`Linq`中的查询，都是延迟执行。

```
var ls=ArticleServices.GetAllArtices().Where(x => x.Id == id);
//ls只有在For-Each遍历时才真正执行where
```
而 `List<T> .FindAll(Predicate<T> match)` 获取查询结果，在表现形式和 `Where` 是一样的。但是 `FindAll` 是顺序执行。

例如：

>在这里查看[完整示例代码](https://github.com/haifengwang/LazyExplore)

```
//用FindAll获取数据
private static void LazyFindExcute()
        {
            User blogUser = new User(1);
            var list = blogUser.Articles;
            var findLs = blogUser.Articles.FindAll(x => x.Id > 1);
            list.Add(new Article { Id = 5, Title = "后添加", PublishDate = DateTime.Now });

            foreach (var item in findLs)
            {
                Console.WriteLine(item.ToString());
            }
        }
        
```
输入结果:

```
----------------Find查询不延迟执行 ------------------------
Article Initalizer
User Initializer 未使用任何
Id=2,Title=Delegate,PublishDate=2015-04-21
Id=3,Title=Event,PublishDate=2015-04-22
Id=4,Title=Thread,PublishDate=2015-04-23
```
用 `Where` 实现以上查询 

```
 private static void LazyWhereExcute() 
        {
            User blogUser = new User(1);
            var list = blogUser.Articles;
            var whereLs = blogUser.Articles.Where(x => x.Id > 1);
            list.Add( new Article{Id=5,Title="后添加",PublishDate=DateTime.Now});

            foreach (var item in whereLs)
            {
                Console.WriteLine(item.ToString());
            }
        }
```

输出结果:

```
----------------where 延迟------------------------
Article Initalizer
User Initializer 未使用任何
Id=2,Title=Delegate,PublishDate=2015-04-21
Id=3,Title=Event,PublishDate=2015-04-22
Id=4,Title=Thread,PublishDate=2015-04-23
Id=5,Title=后添加,PublishDate=2015-07-11
```
执行结果显示，后添加的内容在用 `linq` 查询，遍历时展现结果。

-----

为什么在 `Linq` 中实现了延迟呢？不放看看 `Where` 其中一个最原始的实现。

```
internal static IEnumerable<T> Where<T>(this IEnumerable<T> enumerable, Func<T, bool> where) {
            foreach (T t in enumerable) {
                if (where(t)) {
                    yield return t;
                }
            }
        }
```
在这里发现了 **yield** :

`yield` 关键字形成 `iterator block` ，便有了延迟执行的效果。大部分的 `LINQ to Objects API` ，也就是 `Enumerable static class` 上，几乎都是 `IEnumerable<TSource>` 基础上的扩展。

**yield**在 `.NET` 中被发扬光大，利用它可以做很多很酷的事情。 **<<C# in Depth>>** 中对 `yield`从原理到应用做了很深的探讨，可以[查看这里](http://csharpindepth.com/articles/chapter6/iteratorblockimplementation.aspx)。

利用 `yield` 实现一个延迟示例。示例中包含 `User`类，`Article`类和 `ArticleServices`服务类，[具体代码](https://github.com/haifengwang/LazyExplore)。

```
public static IEnumerable<Article> GetAllArtices() 
        {
            List<Article> articles = new List<Article>
            {
                new Article{Id=1,Title="Lazy Load",PublishDate=DateTime.Parse("2015-4-20")},
                new Article{Id=2,Title="Delegate",PublishDate=DateTime.Parse("2015-4-21")},
                new Article{Id=3,Title="Event",PublishDate=DateTime.Parse("2015-4-22")},
                new Article{Id=4,Title="Thread",PublishDate=DateTime.Parse("2015-4-23")}
            };
            Console.WriteLine("文章初始化完成");
            foreach (var item in articles)
            {
                yield return item;
            }
            Console.WriteLine("Article Initalizer");
            
        }
        
```
在用户类中：

```
public  class User
    {
       public int Id { get; private set; }

        public List<Article> Articles { get; private set; }

        public IEnumerable<Article> OneArtice { get; private set; }

        public User(int id)
        {
            this.Id = id;
            Articles = ArticleServices.GetArtices();
            Console.WriteLine("User Initializer 未使用任何");
        }
        public User(int id,bool isOk)
        {        
            this.Id = id;
            //OneArtice 初始化会延迟到OneArtice调用进行
            OneArtice = ArticleServices.GetAllArtices();
            
            Console.WriteLine("User Initializer 另一种延迟");
        }
    }

```
----
## System.Lazy<T>

通过以上的代码实现了初始化的延迟。但是以上初始化的延迟耦合到了对象集合构建过程中 `GetAllArtices（）`。因而在 `.NET4.0` 中引入了 `System.Lazy<T>`。

**System.Lazy<T>** 提供丰富的延迟方式。[具体查看](https://msdn.microsoft.com/zh-cn/library/dd642331\(v=vs.100\).aspx)

以下通过 `Lazy`改写 `User` 类。

```
public class User_lazy
    {
       public int Id { get; private set; }

        public Lazy<List<Article>> Articles { get; private set; }

        public User_lazy(int id)
        {
            this.Id = id;
            Articles =new Lazy<List<Article>>(()=>ArticleServices.GetArtices());
            Console.WriteLine("User_lazy Initializer");
        }
    }
```

`GetArtices`的实现，将是一个自然顺序的实现。

```
public static List<Article> GetArtices()
        {
            List<Article> articles = new List<Article>
            {
                new Article{Id=1,Title="Lazy Load",PublishDate=DateTime.Parse("2015-4-20")},
                new Article{Id=2,Title="Delegate",PublishDate=DateTime.Parse("2015-4-21")},
                new Article{Id=3,Title="Event",PublishDate=DateTime.Parse("2015-4-22")},
                new Article{Id=4,Title="Thread",PublishDate=DateTime.Parse("2015-4-23")}
            };
            Console.WriteLine("Article Initalizer");
            return articles;
        }
```
##总结

`Deferred Execution`在多核时代，成为了各个高级语言的重要特性。应用场景还是蛮多的。比如说创建一个对象，需要包含子对象，但子对象的并不需要及时初始化；比如说一个大对象中有多个子对象，比如说要优化程序的启动速度等。最常见的示例恐怕是各种 `ORM` 框架了。记着以后自己写访问类，记得一定要支持 `Deferred Execution`。
