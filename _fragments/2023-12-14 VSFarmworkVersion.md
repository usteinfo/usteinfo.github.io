---
layout: fragment
title: Visual Studio 2022如何支持低版本.NET框架
tags: [VisualStudio, .NET]
description: some word here
keywords: VisualStudio, .NET
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

用VS2013开发的一些.NET4、.NET4.5的项目，现在用VS2022打开基本都是让升级，不升级去下载旧版本的话，会发现安装不了......

为了解决这个问题以前是让把VS2013再装回去，然后可以在VS2022中同步使用这些框架，不过近来发现大家换了一种新的方法，直接把包放进去就行了，操作步骤如下。

此处以.NETFramework4.5为例，首先去[Nuget官网](https://www.nuget.org/packages),搜索NET45，会找到这个包

```
Microsoft.NETFramework.ReferenceAssemblies.net45
```
点进去之后，在右边可以看到 [Download package](https://www.nuget.org/api/v2/package/Microsoft.NETFramework.ReferenceAssemblies.net45/1.0.3)，点击下载就行

下载下来后是一个nupkg文件(实际为一个zip格式压缩文件)，可以通过压缩软件的方式打开，复制或者解压目录：
```
\build\.NETFramework\v4.5
```
然后将这个文件夹复制、替换到目录
```
C:\Program Files (x86)\Reference Assemblies\Microsoft\Framework\.NETFramework
```

最后，重启VS2022，就可以在VS2022的项目框架中看到.NET4.5的选项了。

常用项目框架下载路径

[.net 45](https://www.nuget.org/api/v2/package/Microsoft.NETFramework.ReferenceAssemblies.net45/1.0.3)
[.net 451](https://www.nuget.org/api/v2/package/Microsoft.NETFramework.ReferenceAssemblies.net451/1.0.3)
[.net 40](https://www.nuget.org/api/v2/package/Microsoft.NETFramework.ReferenceAssemblies.net40/1.0.3)
[.net 452](https://www.nuget.org/api/v2/package/Microsoft.NETFramework.ReferenceAssemblies.net452/1.0.3)
[.net 20](https://www.nuget.org/api/v2/package/Microsoft.NETFramework.ReferenceAssemblies.net20/1.0.3)
[.net 35](https://www.nuget.org/api/v2/package/Microsoft.NETFramework.ReferenceAssemblies.net35/1.0.3)
