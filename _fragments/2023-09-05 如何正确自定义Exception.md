---
layout: fragment
title: 如何正确自定义 Exception
tags: [c#, net]
description: 自定义异常，除了满足基本信息的存储，还要考虑对象序列化和反序列化的兼容性
keywords: Exception, C#,net
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

在应用程序开发的过程中，错误的处理，有两种方式：
- 返回错误编码
- 抛出异常

在C++中，采用的就是按返回错误码的方式，如：windows win32 api基本都是返回0表示成功，其他表示失败；C#中主要是抛出异常的方式来，传递错误信息，同时抛出的异常必须有相应的处理业务逻辑，还可以避免错误码被忽略的场景。

.net中提供很多系统预定义异常，方便应用的开发，但还是有部分场景需要自定义异常的，如：
- 开发应用框架
- 开发应用项目

自定义异常，除了满足基本信息的存储，还要考虑对象序列化和反序列化的兼容性，有如下要求：
- 异常类名称一定要以后缀 Exception 结尾
- 实现序列化接口
- 从系统异常`Exception`进行继承
- 实现默认四种构造函数，并调用对应基类的构造
- 有自定义属性时，还需要重写方法：GetObjectData

示例：
```csharp
    [Serializable]
    public class ToolException : Exception
    {
        public string ErrorCode { get; }

        public ToolException()
        {
        }

        public ToolException(string message, string errorCode) : base(message)
        {
            ErrorCode = errorCode;
        }

        public ToolException(string message, Exception inner): base(message, inner)
        {
        }

        protected ToolException(SerializationInfo info, StreamingContext context): base(info, context)
        {
            ErrorCode = info.GetString("ErrorCode");
        }

        public override void GetObjectData(SerializationInfo info, StreamingContext context)
        {
            if (!string.IsNullOrEmpty(ErrorCode))
            {
                info.AddValue("ErrorCode", ErrorCode);
            }
            base.GetObjectData(info, context);
        }
    }
```


#### 参考资料
[1、how-to-create-user-defined-exceptions](https://learn.microsoft.com/en-us/dotnet/standard/exceptions/how-to-create-user-defined-exceptions)

[2、best-practices-for-exceptions](https://learn.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions)
