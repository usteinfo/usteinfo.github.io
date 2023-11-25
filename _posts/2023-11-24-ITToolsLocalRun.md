---
layout: post
title: Windows本地运行 ITTools
categories: [ITTools, .Net]
description: some word here
keywords: ITTools, .Net
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

[ITTools](https://github.com/CorentinTh/it-tools) 是一个综合开发工具，提供了docker的运行模式，在windows下没有相关的自运行方式。采用.net core做扩展的应用程序，在本地自运行此工具。


## 下载ITTools源码

从 github [复制](https://github.com/CorentinTh/it-tools.git) 官方源码，本地安装好node和npm,

```
npm install
npm run build
```

## 准备运行环境
ITTools使用vue和typescript开发，需要node和npm环境

```powershell
PS C:\MyData\Source\it-tools-master> node --version
v18.16.1
PS C:\MyData\Source\it-tools-master> npm --version
9.5.1
```

## 打包ITTools

解压源码目录，shell中进入此目录后按序执行

```
# 安装依赖包
npm install

# 文件打包
npm rum build
```

打包好的文件在：dist目录

```
PS C:\MyData\Source\it-tools-master\dist> dir

    目录: C:\MyData\Source\it-tools-master\dist

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        2023/11/25     11:27                css
d-----        2023/11/25     11:27                fonts
d-----        2023/11/25     11:27                img
d-----        2023/11/25     11:27                js
-a----        2023/11/25     11:27            266 browserconfig.xml
-a----        2023/11/25     11:27          15086 favicon.ico
-a----        2023/11/25     11:27            101 humans.txt
-a----        2023/11/25     11:27           4661 index.html
-a----        2023/11/25     11:27            317 manifest.json
-a----        2023/11/25     11:27           9010 precache-manifest.5928982fbda43f57074e8bc65f8a8d7c.js
-a----        2023/11/25     11:27             28 robots.txt
-a----        2023/11/25     11:27            985 service-worker.js
```

编译错误：

- Building for production...Error: error:0308010C:digital envelope routines::unsupported
    
    修改package.json，在相关构建命令之前加入SET NODE_OPTIONS=--openssl-legacy-provider
    
    ```json
    "scripts": {
       "serve": "SET NODE_OPTIONS=--openssl-legacy-provider && vue-cli-service serve",
       "build": "SET NODE_OPTIONS=--openssl-legacy-provider && vue-cli-service build"
    },
    ```
    调整方法引用来自引用，[查看](https://blog.csdn.net/fengyuyeguirenenen/article/details/128319228)

## .Net Core站点包装程序

建立.net core web应用程序，在项目中新建目录“wwwroot”,再把dist目录中的文件拷贝到此目录。

打开program.cs文件，编辑内容如下：

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);
        
        var app = builder.Build();

        app.UseDefaultFiles();
        app.UseStaticFiles();

        app.Run();
    }
}
```

编译运行程序,输出

```
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        2023/11/25     10:44                wwwroot
-a----        2023/11/24     22:24            127 appsettings.Development.json
-a----        2023/11/24     22:24            151 appsettings.json
-a----        2023/11/24     22:38            425 ITToolsWarp.deps.json
-a----        2023/11/24     22:40           5120 ITToolsWarp.dll
-a----        2023/11/24     22:40         140800 ITToolsWarp.exe
-a----        2023/11/24     22:40          20320 ITToolsWarp.pdb
-a----        2023/11/24     22:38            416 ITToolsWarp.runtimeconfig.json
-a----        2023/11/24     22:38          35458 ITToolsWarp.staticwebassets.runtime.json
```

[下载工具](/assets/file/ittools.zip)

运行

```
ITToolsWarp.exe

info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5000
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Production
      
```

### 站点目录变更方法

```csharp
namespace FirstCoreWebApplication
{
    public class Program
    {
        public static void Main(string[] args)
        {
            WebApplicationBuilder builder = WebApplication.CreateBuilder(new WebApplicationOptions
            {
                WebRootPath = "wwwroot"
            });

            var app = builder.Build();
            
            app.UseStaticFiles();
            
            app.Run();
        }
    }
}
```
