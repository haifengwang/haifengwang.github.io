---
layout: post
title: 略谈 .NET 中 通过 Parallel 并行编程
category: net
description: 在 .NET 4.0 引入了 TPL ，对多线程做了更便捷的扩展。同时 Parallel 支持多核编程。
keywords: Parallel,Parallel.For,多线程,线程池,TPL 
--- 

## 背景

在如今的大数据时代，多核 cpu 的发展，如何利用多核，提升计算能力，成为程序开发中一个很重要的话题。随之衍生了专为多核而生的语言，比如说 `Golang` 和 `Erlang` 。曾浮光掠影的看过 `Golang` 的多核编程，关于它的取得 cpu 核数，背后的概念而感叹。而 `Csharp` 作为一个一直走在时代前沿的语言，在 `.NET 4.0` 后引入了 `System.Threading.Tasks` 提供了对多核的支持。

## 场景

曾经在工作中经常通过读取数据，生成静态的 txt 文件。在大批量生成的时，曾为效率问题而头疼过。

刚开始的做法是这样的：

```
public static void CommomCreateTxt()
        {
            var ls = GetPersonLs();
            Stopwatch watch = new Stopwatch();
            watch.Start();
            foreach (var item in ls)
            {
                File.WriteAllText(string.Format(@"F:\project\teststatic\{0}.txt", item.Id), item.ToString(), Encoding.UTF8);
            }
            watch.Stop();
            Console.WriteLine("普通循环生成时间：{0}", watch.ElapsedMilliseconds.ToString());
            //423 440 
        }
        
```
这种很原始的用顺序执行的方式生成静态文件。不过效率确实有问题。

于是想到利用线程，但如何在循环中更有效率的使用线程呢？答案是**线程池**。

```
public static void ThreadPoolCreateTxt()
       {
           var ls = GetPersonLs();
           Stopwatch watch = new Stopwatch();
           watch.Start();

           ManualResetEvent eventX = new ManualResetEvent(false);
           //可以获取和设置线程池的最大线程数，即线程池中共有多少个线程。线程池中的线程数目仅受可用内存的限制。但是，线程池将对允许在进程中同时处于活动状态的线程数目强制实施限制（这取决于CPU的数目和其他因素）。默认情况下，每个系统处理器最多可以运行25个线程池线程。
           ThreadPool.SetMaxThreads(3, 3);

           foreach (var item in ls)
           {
               ThreadPool.QueueUserWorkItem(x => {
                   File.WriteAllText(string.Format(@"F:\project\teststatic\{0}.txt", item.Id), item.ToString(), Encoding.UTF8);
               });
               
           }
           watch.Stop();
           Console.WriteLine("线程池生成时间：{0}", watch.ElapsedMilliseconds.ToString());
       }
```

这种做法效率明显提升了，也达到了当时的需求。

## Parallel Class

.NET4.0 在 **mscorlib** 增加了 `System.Threading.Tasks`,旨在在简化多线操作的不方便性，同时利用多核的优势，进行并行计算。

`System.Threading.Tasks.Parallel` 中有有 `For` 、`ForEach` 和 `Invoke`  提供了多种方式的重载，提供了很大的便利性。

程序被改进为：

```
public static void ParallelCreateTxt()
        {
            var ls = GetPersonLs();
            Stopwatch watch = new Stopwatch();
            watch.Start();
            Parallel.ForEach(ls, new ParallelOptions{MaxDegreeOfParallelism=Environment.ProcessorCount}, item => { File.WriteAllText(string.Format(@"F:\project\teststatic\{0}.txt", item.Id), item.ToString(), Encoding.UTF8); });

            watch.Stop();
            Console.WriteLine("Parallel循环生成时间：{0}", watch.ElapsedMilliseconds.ToString());
            
        }
```
代码很简洁，效率还可以。

## 效率

用 `Parallel.ForEach` 后效率比普通的高，但是并不优于 `ThreadPool`.于是沿着这个疑问，开始搜寻,得到以下信息。

+ `Parallel.ForEach` 在内部，采用了 `Partitioner<T>` 分发收集到的工作项。它不会对每一项操作，而是针对一批认为这种降低所涉及的开销。

+ **TPL** 有一个像简单的异常处理，取消支持，能够方便地返回任务结果的优点。

+ `Parallel.For` 功能是基于 TPL 任务。任务所基于的线程池。

可以看到线程池更基础，那问题是多核的优势哪里去了？

在 github 上找到了一个[这样的测试](https://github.com/raistlinthewiz/concurrency-tests)。

根据作者的观点，`Parallel` 并没有多少性能的提升，而是在操作的便捷性上了作出了好多扩展。提供了简单的异常处理，取消支持，能够方便地返回任务结果等优点。

鉴于疑问，找到了[Parallel For Loop Processing With C# and .NET](http://www.kobashicomputing.com/parallel-for-loop-processing-with-c-and-net) 。

提到：
>Parallel loop processing is ideal when you are iterating over a collection of immutable objects. The object states are independent of any processing iterating through the collection. That is, you can change the values of the object properties and it doesn't affect the processing behavior.

>Whenever you make a parallel processing structure, think of it as request to do parallel processing and not a guarantee that it will ever occur. Also, unlike the sequential processing case where the iterator goes from 1 to N, in the parallel loop case, the iterator can start anywhere and be of any value (not sequential).

大意就是并行是一种理想的状态，需要一种并行的结构做支撑。

好吧，探索到最后，我的程序能利用的还是多线程。

完整代码片段  [https://gist.github.com/haifengwang/34b8408f7bd92a2b5ffd](https://gist.github.com/haifengwang/34b8408f7bd92a2b5ffd)