---
layout: fragment
title: 如何在 await,async 中使用 lock 锁 ？
tags: [net, await,async,lock]
description: 
keywords: net, await,async,lock
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

在异步中使用lock，会出现如下错误

```
error CS1996: Cannot await in the body of a lock statement
```

要解决此问题，有两个办法：
- 使用 C# 新提供的 `SemaphoreSlim.WaitAsync` 方法

```csharp
private readonly SemaphoreSlim readLock = new SemaphoreSlim(1, 1); 

public async Task UpdateDetailsAsync()
{
    await readLock.WaitAsync();
    try
    {
        Details details = await connection.GetDetailsAsync();
        detailsListBox.Items = details;
    }
    finally
    {
        readLock.Release();
    }
}
```

- 使用`Nito.AsyncEx`库

它是一个 Nuget 上非常有名的异步扩展库，很多的开发者也都是从用 AsyncLock 开始的，使用方式和同步模式的 lock 很相似，参考如下代码：

```csharp
private readonly AsyncLock _mutex = new AsyncLock();

public async Task UseLockAsync()
{
  // AsyncLock can be locked asynchronously
  using (await _mutex.LockAsync())
  {
    // It's safe to await while the lock is held
    await Task.Delay(TimeSpan.FromSeconds(1));
  }
}
```

更多的内容，可以参考 [github](https://github.com/StephenCleary/AsyncEx)
