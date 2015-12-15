---
layout: post
title: ASP.NET Web API 中使用 Cookie 和 Session 
category: net
description: 最近接到一个需求:需要在 ASP.NET Web API 中对一个外部 API 做一次调用。而外部 API 又是以 Cookie 作为验证。问题来了,在 ASP.NET Web API 中当然是不能记录 Cookie 那怎么办？
keywords: ASP.NET,Web API,Cookie,Session
--- 


##背景
最近接到一个需求:需要在 `ASP.NET Web API` 中对一个外部 `API` 做一次调用。而外部 `API` 又是以 `Cookie` 作为验证。问题来了,在 ASP.NET Web API 中当然是不能记录 `Cookie` ，但 `Cookie` 还得记下该怎么办？

解决办法:通过第三方存储:如文件、缓存、数据库等。我选择较为方便的 `Session`.

##实现

###第一步:登录

通过 `HttpClient` 登录,获取 `Cookie` 并保存到 `Session` 中

```
    HttpClient client = new HttpClient();
    //地址是随便写的
    client.BaseAddress = new Uri("http://api.com/"); 
    HttpResponseMessage response = client.GetAsync("api/login").Result;
            if (response.IsSuccessStatusCode)
            {
                 var resbody = response.Content.ReadAsStringAsync().Result;

                    var headers = response.Headers;
                    IEnumerable<string> cookies = null;
                    //在相应头中获取 Cookie 
                    if (headers.TryGetValues("Set-Cookie", out cookies))
                    {
                        String[] fields = null;
                        foreach (var item in cookies)
                        {
                            fields = Regex.Split(item, ";\\s*");
                        }
                        //fields.Leng=3,分别为 值、域、过期时间
                        if (fields != null)
                        {
                            string cookie = fields[0];
                            if (!string.IsNullOrEmpty(cookie))
                            {

                               //获取到的 cookie 值是这样的 cookieName=xxxxxxxxxxxxx
                                    HttpContext.Current.Session["LoginSession"] =cookie.Substring("CookieName".Length+1);              
                                
                            }
                            else 
                            {
                                throw new Exception("登录失败");
                            }
                        }

                    }

            }
```
### 第二步:携带 Cookie 请求接口
携带 Cookie 信息时需要用到 `HttpClientHandler` 
+ 构建带有 `Cookie` 的 `HttpClientHandler`

```
string yCookie = string.Empty;
           
            yCookie = HttpContext.Current.Session[SESSIONKEY] as string;
            //这里省去判断 Session 为空的操作
            CookieContainer cookieContainer = new CookieContainer();
            //注意这里的 Cookie 需要全构造，不能有空值出现，否则验证不通过
            Cookie cookieTarget = new Cookie()
            {
                Domain = Host,
                Name = "cookieName",
                Path = "/",
                Secure = false,
                Value = yCookie
            };

            cookieContainer.Add(cookieTarget);
            HttpClientHandler handler = new HttpClientHandler
            {
                UseCookies = true,
                UseDefaultCredentials = true,
                CookieContainer = cookieContainer
            };
```
+ 请求

```
 HttpClient client = new HttpClient(handler);

client.BaseAddress = "http://api.com/";
HttpResponseMessage response =
                    client.GetAsync("api/getInfo").Result;
```

这样就基本完成了这个需求。

## 知识点
+ **HttpClinet** 

`HttpClinet` 是 `.NET 4.5` 引入在 `System.Net.Http.HttpClient` 下 ,作为 Http 请求和响应的基类。

看到 `HttpClinet` 也许你会想到它一个兄弟 `WebClient`。`WebCliebt` 在 `.NET 2.0` 中引入，提供向 URI 标识的资源发送数据和从 URI 标识的资源接收数据的公共方法。

问题：它们两个有什么异同？

  **HttpClient** 更接近 Http ,基于 **Henrik F Nielson** 标准设计，基于它设计 API 很容易的遵循 HTPP 标准。利用了异步编程模型设计；非常实用于 WebService 和 RESTFull API。
  
  **WebClient** 是 `.NET 2.0` 提供的，支持同步、支持下载进度、支持FTP。
  
  总结：`HttpClient` 比 `WebClient` 更贴近 `HTTP` 。 但并不意味着完全取代 `WebClient`，例如事件进展报告，自定义的URI方案，支持FTP。这些 `HttpClient` 不能直接做的。

+ **HttpClientHandler**

`HttpClienHanlder` 是一个常见的代理模式的设计，它在 `HttpClient.GetStringAsync()` 中加了一层封装，拦截了 `HttpClient` 的输入和输出，从而实现一些自定义的操作。

通常在应用中继承 `HttpClienHanlder` ,重写 `SendAsync` 完成自定义操作。

```
protected override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, System.Threading.CancellationToken cancellationToken)
        {
            //添加自定内容
            return base.SendAsync(request, cancellationToken);
        }
```
+ **ASP.NET Web API** 中使用 `Session` 

`Web API` 默认不支持 `Session` ,需要在 `Global.asax` 文件中重写 `Init` 方法开启。 

```
public override void Init()
        {
           this.PostAuthenticateRequest += (sender, e) => HttpContext.Current.SetSessionStateBehavior(SessionStateBehavior.Required);
            base.Init();
        }

```
这样简单粗暴打开全局的 `Session`。如果只是某一个路由需要 `Session` 呢？在这个需求中也仅仅是调用外部 API 的这个路由需要。

另一种方式针对每一个路由。创建一个实现 IRequiredSessionState 的 Handler。将 Handler 附加到 Web API 的路由，连接到 Web 的 API 的路由会根据要求使会话状态。

第一步：继承 `HttpControllerRouteHandler` ,创建 `SessionHttpControllerRouteHandler`

```
public class SessionHttpControllerRouteHandler : HttpControllerRouteHandler  
    {  
        protected override IHttpHandler GetHttpHandler(RequestContext requestContext)  
        {  
            //将路由转接都另一个 Controller
            return new SessionControllerHandler(requestContext.RouteData);  
        }  
    }
```
第二步：创建 `SessionControllerHandler` ,继承 `HttpControllerHandler` 实现 `IRequiresSessionState`,从而开启了 Session

```
public class SessionControllerHandler : HttpControllerHandler, IRequiresSessionState  
    {  
        public SessionControllerHandler(RouteData routeData)  
            : base(routeData)  
        { }  
    } 
```
第三步：在 System.Web.RouteCollection 注册 Web API 路由

```
routes.MapHttpRoute(  
                name: "DefaultApi",  
                routeTemplate: "api/{controller}/{id}",  
                defaults: new { id = RouteParameter.Optional }  
                ).RouteHandler = new SessionHttpControllerRouteHandler();  
   
```

##参考资料

[ASP.NET Web API](http://www.asp.net/web-api)

[WebClient Vs HttpClient](http://stackoverflow.com/questions/20530152/need-help-deciding-between-httpclient-and-webclient)