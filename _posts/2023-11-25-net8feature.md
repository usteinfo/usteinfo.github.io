---
layout: post
title: .Net8新特性
categories: [Net, 新特性]
description: 本文整理.Net8新特性的使用方法
keywords: Net, 新特性
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

本文整理.Net8新特性的使用方法。
当前包括：
- Route ShortCircuit
- Exception Throw Helper
- HttpLoggingMiddleware 的改进
- C# 12 中的 InlineArray 特性
- 随机数增强
- KeyedServices
- IPNetwork

## Route ShortCircuit

有些请求，如浏览器会自动请求 favicon.ico，这些请求即使很简单，往往也会完整地运行中间件管道，但实际上可能并不需要，在 .NET 8 中引入了一个 Route ShortCircuit 的功能，也就是路由短路，可以在处理结束之后马上中断请求，不再执行后面的中间件了，这样会使得这样的路由或者 API 更加高效。

在 route 后添加 `ShortCircut()` 来启用，使用方法如下：

```csharp
var builder = WebApplication.CreateSlimBuilder(args);

var app = builder.Build();
app.Use(async (HttpContext context, RequestDelegate next) =>
{
    context.Response.Headers["Value"] = "123";
    await next(context);
});
app.MapGet("/", () => "Hello .NET 8!");

// ShortCircuit
app.MapGet("/short-circuit", () => "Short circuiting!").ShortCircuit();
app.MapGet("/short-circuit-status", () => "Short circuiting!")
            .ShortCircuit(401);
// MapShortCircuit
app.MapShortCircuit(404, "robots.txt", "favicon.ico");
app.MapShortCircuit(403, "admin");

await app.RunAsync();
```

## Exception Throw Helper

在 .NET 6 中，引入了一个 ArgumentNullException.ThrowIfNull(object? argument, string? paramName = default) 的方法，在 .NET 7/8 中引入了更多的支持，我们可以在代码里使用这些 exception helper 来简化一些代码。

常用的 Argument exception：

```csharp
ArgumentNullExceptionSample(null);

ArgumentExceptionThrowIfNullOrEmptySample(null);
ArgumentExceptionThrowIfNullOrEmptySample(string.Empty);

ArgumentExceptionThrowIfNullOrWhiteSpaceSample(null);
ArgumentExceptionThrowIfNullOrWhiteSpaceSample(string.Empty);
ArgumentExceptionThrowIfNullOrWhiteSpaceSample(" ");

public static void ArgumentNullExceptionSample(string? value)
{
    InvokeHelper.TryInvoke(() => ArgumentNullException.ThrowIfNull(value));
}

public static void ArgumentExceptionThrowIfNullOrEmptySample(string? value)
{
    InvokeHelper.TryInvoke(() => ArgumentException.ThrowIfNullOrEmpty(value));
}

public static void ArgumentExceptionThrowIfNullOrWhiteSpaceSample(string? value)
{
    InvokeHelper.TryInvoke(() => ArgumentException.ThrowIfNullOrWhiteSpace(value));
}
```

`ArgumentNullException.ThrowIfNull(object? obj, string? paramName = default)`  是 .NET 6 开始支持的

在 .NET 7 里支持了指针的判断 `ArgumentNullException.ThrowIfNull(void* argument, string? paramName = default)`(not CLS-compliant)

.NET 7 还引入了 `ArgumentException.ThrowIfNullOrEmpty(string? value, string? paramName = default)`, 增加判断空字符串的场景

.NET 8 引入了 `ArgumentException.ThrowIfNullOrWhiteSpace(string? value, string? paramName = default)` 增强了判断空字符串的场景

##  HttpLoggingMiddleware 的改进

.NET 6 开始引入了一个 http logging 的中间件，我们可以借助于 http logging 的中间件记录请求和响应的信息，但是扩展性不是很强，在 .NET 8 版本中进行了一些优化，引入了一些新的配置和 HttpLoggingInterceptor 使得它更加容易扩展了

HttpLoggingFields 中新增了一个 Duration 枚举值，会记录请求处理的耗时

在 HttpLoggingOptions 中增加了一个 CombineLogs 的配置，默认是 false，默认 request/response/duration 的 log 都是分开的,配置为 true 之后就会合并成一条日志。

##　HttpLoggingInterceptor

.NET 8 还引入了 IHttpLoggingInterceptor，借助于此可以更好的扩展 http logging

可以根据 Request 或者 Response 信息来动态地调整要记录的 field 或者动态调整 RequestBodyLogLimit/ResponseBodyLogLimit

来看一个 HttpLoggingInterceptor 示例：

```csharp
file sealed class MyHttpLoggingInterceptor: IHttpLoggingInterceptor
{
    public ValueTask OnRequestAsync(HttpLoggingInterceptorContext logContext)
    {
        if (logContext.HttpContext.Request.Path.Value?.StartsWith("/req-") == true)
        {
            logContext.LoggingFields = HttpLoggingFields.ResponsePropertiesAndHeaders;
            logContext.AddParameter("req-path", logContext.HttpContext.Request.Path.Value);
        }
        
        return ValueTask.CompletedTask;
    }

    public ValueTask OnResponseAsync(HttpLoggingInterceptorContext logContext)
    {
        if (logContext.HttpContext is { Response.StatusCode: >=200 and < 300, Request.Path.Value: "/hello" })
        {
            logContext.TryDisable(HttpLoggingFields.All);
        }
        return ValueTask.CompletedTask;
    }
}
```

使用示例如下，使用 AddHttpLoggingInterceptor<TInterceptor>() 来注册：

```csharp
var builder = WebApplication.CreateSlimBuilder(args);
builder.Services.AddHttpLogging(options =>
{
    options.LoggingFields = HttpLoggingFields.All;
    options.CombineLogs = true;
});
builder.Services.AddHttpLoggingInterceptor<MyHttpLoggingInterceptor>();

var app = builder.Build();
app.UseHttpLogging();
app.MapGet("/hello", () => "Hello");
app.MapGet("/crash", () => Results.BadRequest());
app.MapGet("/req-intercept", () => "Hello .NET 8");
app.Run();
```

appliction.json文件中加入

```
"Microsoft.AspNetCore.HttpLogging.HttpLoggingMiddleware": "Information"
```

运行应用


```
PS C:\MyData\Source\Net8Test\WebApplication1\bin\Debug\net8.0> dotnet-http :5055/hello
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
Date: Sat, 25 Nov 2023 08:53:59 GMT
Server: Kestrel
Transfer-Encoding: chunked

//日志

info: Microsoft.AspNetCore.HttpLogging.HttpLoggingMiddleware[9]
      Request and Response:
      Protocol: HTTP/1.1
      Method: GET
      Scheme: http
      PathBase:
      Path: /hello
      Host: localhost:5055
      User-Agent: dotnet-httpie/0.7.2
```

```
PS C:\MyData\Source\Net8Test\WebApplication1\bin\Debug\net8.0> dotnet-http :5055/crash
HTTP/1.1 400 BadRequest
Content-Length: 0
Date: Sat, 25 Nov 2023 08:53:40 GMT
Server: Kestrel

info: Microsoft.AspNetCore.HttpLogging.HttpLoggingMiddleware[9]
      Request and Response:
      Protocol: HTTP/1.1
      Method: GET
      Scheme: http
      PathBase:
      Path: /crash
      Host: localhost:5055
      User-Agent: dotnet-httpie/0.7.2
      StatusCode: 400
      Duration: 9.8983
```

```
PS C:\MyData\Source\Net8Test\WebApplication1\bin\Debug\net8.0> dotnet-http :5055/req-intercept
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
Date: Sat, 25 Nov 2023 08:53:14 GMT
Server: Kestrel
Transfer-Encoding: chunked

info: Microsoft.AspNetCore.HttpLogging.HttpLoggingMiddleware[9]
      Request and Response:
      req-path: /req-intercept
      StatusCode: 200
      Content-Type: text/plain; charset=utf-8
```


可以看到每个请求的 log 输出的结果都有所不同，第一个请求虽然我们设置了 ogContext.TryDisable(HttpLoggingFields.All) 但是还是有输出结果这是因为 httpLogging 目前的实现就是这样，在 Response 里处理的时候 request 信息已经被记录好了，详细可以参考 http logging middleware 的 [实现](https://github.com/dotnet/aspnetcore/blob/main/src/Middleware/HttpLogging/src/HttpLoggingMiddleware.cs)

如果想要完全 disable 需要在 OnRequestAsync 方法里处理

```csharp
public ValueTask OnRequestAsync(HttpLoggingInterceptorContext logContext)
{
    if ("/no-log".Equals(logContext.HttpContext.Request.Path.Value, StringComparison.OrdinalIgnoreCase))
    {
        logContext.LoggingFields = HttpLoggingFields.None;
    }
    //
    return ValueTask.CompletedTask;
}
```

这样请求就不会有日志打印了

最后一个 req-intercept 在  request 的处理中设置了 ResponsePropertiesAndHeaders 并且加了一个自定义的 Parameter 从输出结果可以看到有输出到日志

More
大家可以自己尝试一下，比之前会好用一些，但是觉得还是有所欠缺

比如日志级别目前还都是 Information 不能动态的改变日志级别

另外就是前面提到的即使使用 CombineLogs 在 response 中设置为 HttpLoggingFields.None 时，依然会记录 request 信息，希望后面还会继续优化一下
>dotnet-http 可以通过如下命令安装：dotnet install dotnet-httpie

## C# 12 中的 InlineArray 特性

C# 12 引入了一个 InlineArray 特性，利用这一特性，我们可以更方便地类数组的结构体，可以代替原来要使用非安全代码的 fixed size buffer

```csharp
[System.Runtime.CompilerServices.InlineArray(10)]
file struct MyArray
{
    // required
    private int _element;
}
```

使用 InlineArray 需要指定 size，也就是 array 的长度，并且我们需要声明一个字段

使用示例如下：

```csharp
var arr = new MyArray();
for (var i = 0; i < 10; i++)
{
    arr[i] = i;
}
foreach (var i in arr)
{
    Console.Write(i);
    Console.Write(",");
}

Console.WriteLine();

ReadOnlySpan<int> span = arr;
foreach (var i in span)
{
    Console.Write(i);
    Console.Write(",");
}

Console.WriteLine();

foreach (var i in arr[^2..])
{
    Console.Write(i);
    Console.Write(",");
}
Console.WriteLine();

Console.WriteLine(arr[^1]);

// error CS0021: Cannot apply indexing with [] to an expression of type 'MyArray'
// if (arr is [0,1,..])
//     Console.WriteLine("StartsWith 0, 1");

if (span is [0,1,..])
    Console.WriteLine("StartsWith 0, 1");

// error CS9174: Cannot initialize type 'MyArray' with a collection expression because the type is not constructible.
// arr = [1, 2, 3, 4, 5];

span = [1,2,3,4,5];
foreach (var item in span)
{
    Console.Write(item);
    Console.Write(",");
}
```

运行结果：

```
0,1,2,3,4,5,6,7,8,9,
0,1,2,3,4,5,6,7,8,9,
8,9,
9
StartsWith 0, 1
1,2,3,4,5,
```

从这个示例可以看得出来，我们可以像使用数组一样使用，同时我们可以直接隐式转换成 Span 和 ReadOnlySpan,
并且可以使用 Index 和 Range 操作符,但是目前暂时不能直接使用集合表达式和 list pattern，但是我们可以转成 span 之后再使用

## 随机数增强

1、在 8 中对随机数类 Random 提供了 GetItems()方法，可以根据指定的数量在提供的一个集合中随机抽取数据项生成一个新的集合：

```
ReadOnlySpan<string> colors = new[]{"Red","Green","Blue","Black"};

string[] t1 = Random.Shared.GetItems(colors, 10);
Console.WriteLine(JsonSerializer.Serialize(t1));

//输出：["Black","Green","Blue","Blue","Green","Blue","Green","Black","Green","Blue"]
//每次都会不一样
Console.ReadKey();
```

2、通过 Random 提供的 Shuffle() 方法，可以将一个集合中的数据项的顺序打乱：

```
string[] colors = new[]{"Red","Green","Blue","Black"};
Random.Shared.Shuffle(colors);

Console.WriteLine(JsonSerializer.Serialize(colors));

Console.ReadKey();
```

## KeyedServices

要实现一个接口多个实现类的注入，还需要写一些额外的代码，比较繁琐。

版本 8 中添加了注入关键字，可以很方便实现，看下面代码：

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddKeyedSingleton<IUser, UserA>("A");
builder.Services.AddKeyedSingleton<IUser, UserB>("B");

var app = builder.Build();

app.MapGet("/user1", ([FromKeyedServices("A")] IUser user) =>
{
    return $"hello , {user?.GetName()}";
});
app.MapGet("/user2", ([FromKeyedServices("B")] IUser user) =>
{
    return $"hello , {user?.GetName()}";
});

app.Run();

internal interface IUser
{
    string GetName();
}
internal class UserA: IUser
{
    public string GetName() => "oec2023";
}
internal class UserB : IUser
{
    public string GetName() => "oec2024";
}

```

## IPNetwork

.NET 8 新增了 IPNetwork 的实现，支持 [CIDR](/wiki/2023-11-25%20CIDR/) 网络格式。

```
var ipNetwork = "198.51.0.0/16";
var network = IPNetwork.Parse(ipNetwork);
var ip = IPAddress.Parse("198.51.250.42");
Console.WriteLine(network.Contains(ip));
```

运行结果：
```
true
```
