---
layout: post
title: 利用 UIWebView 对页面分页显示
category: ios
description: UIWebView 在 ios7.0 增加了属性 paginationMode，结合 pagingEnabled
 可以方便的对Web页面分页显示。
keywords: 开发,ios,UIWebView,webView
--- 

**UIWebView** 在 ios7.0 增加了属性 `paginationMode`，结合 `pagingEnabled`
 可以方便的对Web页面分页显示。
 
 **UIWebPaginationMode** 提供五种方式供方便的选择。

+ UIWebPaginationModeUnpaginated 

+ UIWebPaginationModeLeftToRight  //左右翻页

+ UIWebPaginationModeTopToBottom //默认的方式

+ UIWebPaginationModeBottomToTop

+ UIWebPaginationModeRightToLeft 

##分页显示

通过以下代码将网页按照左右翻页的方式呈现（例如 *网易新闻 iPad 版* ）：

```
    UIWebView *aview=[[UIWebView alloc ]initWithFrame:
    [UIScreen mainScreen].bounds];
    NSURL *url=[NSURL URLWithString:[bookArray objectAtIndex:0]];
    aview.scrollView.alwaysBounceHorizontal = YES;
    aview.scrollView.alwaysBounceVertical = NO;
    aview.scrollView.bounces = YES;
    aview.scrollView.showsVerticalScrollIndicator = NO;
    aview.scrollView.showsHorizontalScrollIndicator = NO;
    //这样超出范围scrollView.frame 也会显示，即整个webView还是会正常显示
    aview.scrollView.clipsToBounds = NO; 
     //如果需要后续其他操作，例如页面显示等需要实现  
    aview.scrollView.delegate = self;
    //设置以后就可以水平滑动
    aview.paginationMode = UIWebPaginationModeLeftToRight; 
    aview.paginationBreakingMode = UIWebPaginationBreakingModeColumn;
    aview.scrollView.pagingEnabled = YES;
    //取决于实现逻辑，如果没有后续操作，不需要实现
    aview.delegate=self; 
```
`pagingEnabled` 隶属于 `UIScrollView`。体系结构如下图 

![webView 体系结构]({{ site:url }}/images/2014/uiwebview.png)

如果未设置`pagingEnabled=YES`,滑动时屏幕上显示的页面并不是一个完整的分割好的页面，这个属性的设置非常**重要**。


##显示页码

要获取页面总分页数，需要实现`UIWebViewDelegate`在 `webViewDidFinishLoad:(UIWebView *)webView` 通过属性`pageCount`(*该属性页是7.0后增加*)

如果需要在分页中显示页面，则需要实现`UIScrollViewDelegate`,在`scrollViewDidScroll:(UIScrollView *)scrollView` 添加逻辑，实现每翻一页显示当前页页码。

```
-(void)scrollViewDidScroll:(UIScrollView *)scrollView
{
   
    CGFloat halfWidth=contentWidth/pageCount;

    CGFloat offsetWidth=scrollView.contentOffset.x;

    int num=ceil((offsetWidth+aview.frame.size.width/2)/halfWidth);

    NSLog(@"当前页=%d,总页数=%d",num,aview.pageCount);
}

```

至此，一个对Web页面进行分页显示，并带有页码的功能基本实现。利用这个分页功能还可以其他方面扩展，比如电子书等。