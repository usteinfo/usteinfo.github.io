---
layout: post
title: EF Core 技巧
categories: [ef, c#]
description: 本文描述 EF Core 常用使用技巧，记录学习点滴
keywords: ef, c#
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
本文描述 EF Core 常用使用技巧,可以作为知识库查询EF的使用方法。

#### 输出查询SQL语句
在Entity Framework的第一个迭代版本中，没有内置的日志记录。但是有`ObjectQuery.ToTraceString()`，这是一种运行时方法，可以动态计算LINQ或Entity SQL查询的SQL，尽管这不是一个很好的日志记录方法，但它毕竟可以输出SQL，即使在今天，也有一些有用的场景。
直到最新版本EF Core 5，该功能才成为EF Core的一部分，并且已重命名为`ToQueryString()`。注意在InMemory程序中无效。

```csharp
var sqlFromQuery=context.People.ToQueryString();
```

#### 日志输出
使用LogTo方法轻松配置DbContext，将.NET日志记录输出。
嗯，我就想看着你，就这样子，简简单单。
EF Core将输出很多事件。分为以下类，这些类从DbCloggerCategory派生。

分类|名称
-|-
变更追踪|ChangeTracking
数据库命令|Database.Command
数据库连接|Database.Connection
数据库事务|Database.Transaction
数据库|Database
基础设施|Infrastructure
数据库挖掘|Migrations
模型验证|Model.Validation
模型|Model
查询|Query
脚手架|Scaffolding
更新|Update

LogTo 的一个参数指定目标为控制台窗口、文件或调试窗口。然后，第二个参数允许您通过.NET LogLevel以及您感兴趣的任何DLoggerCategoy进行筛选。

```csharp
//DbContext将日志输出到控制台，并过滤掉所有DbLoggerCategory类型LogLevel.Information组。
optionsBuilder.UseSqlServer(myConnectionString)
.LogTo(Console.WriteLine,LogLevel.Information);
```

下面一个LogTo方法添加了第三个参数-DbLoggerCatetory数组（仅包含一个数组），以便仅对EF Core的数据库命令进行进一步过滤。
与LogTo方法一起，我添加了EnableSensitiveDataLogging方法以在SQL中显示传入参数。这将捕获所有发送到数据库的SQL：查询，更新，原始SQL甚至通过迁移发送的更改。

```csharp
.LogTo(Console.WriteLine, LogLevel.Information,
    new[]{DbLoggerCategory.Database.Command.Name},
)
.EnableSensitiveDataLogging();
```

上面包含IsDeleted属性的“Person”类型也具有FirstName和LastName属性。这是添加新的Person对象后调用SaveChanges的日志。

```csharp
info: 1/4/2021 17:56:09.935
    RelationalEventId.CommandExecuted[20101]
    (Microsoft.EntityFrameworkCore.Database.Command)

Executed DbCommand (22ms) [Parameters=[ 
    @p0='Julie' (Size = 4000), @p1='False', 
    @p2='Lerman' (Size = 4000)],CommandType='Text',
    CommandTimeout='30']

    SET NOCOUNT ON;
    INSERT INTO [People] ([FirstName],[IsDeleted], [LastName])
    VALUES (@p0, @p1, @p2);
    SELECT [Id]
    FROM [People]
    WHERE @@ROWCOUNT = 1 AND [Id] = scope_identity();
```

>注意顶部的EventId。您甚至可以定义日志记录以使用这些ID过滤特定事件。您还可以过滤出特定的日志类别，并且可以控制格式。在 [网站](https://docs.microsoft.com/zh-cn/ef/core/logging-events-diagnostics/simple-logging) 上查看有关这些各种功能的更多详细信息的文档。
简单日志记录是记录EF Core的高级方法，是的，高级的就是简单的，这就是计算机世界的定义！
您也可以通过直接与Microsoft.Extensions.Logging一起，以对EF Core的日志方式进行更多控制。检查EF Core文档，以获取更多有关使用此更高级用法的 [详细信息](https://docs.microsoft.com/zh-cn/ef/core/logging-events-diagnostics/extensions-logging) 。

#### 响应EF Core 事件

事件名称|版本|说明
-|-|-
ChangeTracker.Tracked|2.1|在DbContext开始跟踪实体时引发
ChangeTracker.StateChanged|2.1|在已跟踪的实体的状态改变时引发
DbContext.SavingChanges|5.0|当上下文将要保存更改时
DbContext.SavedChanges|5.0|保存成功时引发
DbContext.SaveChangesFailed|5.0|保存失败时引发
DbContext.SaveChangesAsync|5.0|异步保存成功时引发

#### 事件计数器访问指标

>EF Core 5利用了.NET Core 3.0中.NET引入的一项很酷的功能-dotnet-counters（https://docs.microsoft.com/zh-cn/dotnet/core/diagnostics/dotnet-counters）。计数器是一个全局命令行工具。您可以使用dotnet CLI安装此工具。

```csharp
dotnet tool install --global dotnet-counters
```

安装完成后，您可以告诉它监视在dotnet环境中运行的进程。您需要提供正在运行的.NET应用程序的进程ID

```csharp
System.Diagnostics.Process.GetCurrentProcess().Id
```

在Visual Studio中，我无法简单地在调试器中调试此值。调试器只会告诉您“此表达式会产生副作用，因此不会被评估。” 因此，我将其嵌入到我的代码中并获得了值313131。

在拥有ID且应用仍在运行的情况下，然后可以触发计数器，开始监视来自Microsoft.EntityFramework命名空间的事件。如下：

```csharp
dotnet counters monitor    Microsoft.EntityFrameworkCore -p 313131
```

然后，当您遍历应用程序时，计数器将显示EF Core统计信息的特定列表

#### 拦截EF Core的数据——拦截器(EF Core 3)

EF Core的拦截器是一项功能，该功能始于EF6，并在EF Core 3中重新引入。EF Core 5中引入了SaveChanges的新拦截器。
共有三种不同的拦截器类来拦截命令，连接和事务，以及用于SaveChanges的新拦截器类。每个类都有自己相关的虚拟方法（和相关对象）。

例如，DbCommandInterceptor公开了ReaderExecuting和ReaderExecutingAsync，它们在命令即将发送到数据库时被触发。

```csharp
public override InterceptionResult<DbDataReader>
   ReaderExecuting(
       DbCommand command,
       CommandEventData eventData,
       InterceptionResult<DbDataReader> result)
   {
       //例如，webmote支持你干点啥?
       return result;
   }
```

它的参数之一是DbCommand，其CommandText属性保存SQL。
如果要修改SQL，添加查询提示或其他任务，则可以更改命令，然后使用新CommandText值的命令将继续进行。
从数据库返回任何结果数据时，将触发ReaderExecuted / Async方法。

```csharp
public override DbDataReader ReaderExecuted(
    DbCommand command,
    CommandExecutedEventData eventData,
    DbDataReader result)
    {
        return base.ReaderExecuted
        (command, eventData, result);
    }
```

例如，在这里您可以捕获DbDataReader，并对该数据进行某些处理，然后再继续执行EF Core实现。


#### 查询拦截
EF Core 公开的 DbCommandInterceptor拦截器提供查询拦截功能，查询拦截是在数据库上执行查询之前插入逻辑，或者在查询执行之后（以及控制返回到调用代码之前）立即插入逻辑的能力。
此功能在现实世界中有多种使用案例：
延长具有某些特征的命令的超时
查询失败并记录异常时诊断信息
当读取到内存的行数超过特定阈值时，记录警告
一个小例子：

```csharp
public class TestQueryInterceptor : DbCommandInterceptor
{
    // runs before a query is executed
    public override InterceptionResult<DbDataReader> ReaderExecuting(DbCommand command, CommandEventData eventData, InterceptionResult<DbDataReader> result)
    {
        command.CommandText += " OPTION (OPTIMIZE FOR UNKNOWN)";   
        command.CommandTimeout = 12345;   
         return result;
    }
    
    // runs after a query is excuted
    public override DbDataReader ReaderExecuted(DbCommand command, CommandExecutedEventData eventData, DbDataReader result)
    {
        if (this.ShouldChangeResult(command, out var changedResult))
        {
            return changedResult;
        }
        
        return result;
    }
}
```

注意： 大多数方法都有同步和异步版本。令人讨厌的是，异步查询仅触发异步方法（反之亦然），因此在编写拦截器时必须覆盖两者。
安装拦截器是很简单的。

```csharp
public class SampleDbContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder
            .UseSqlite(@"Data Source=Sample.db;")
            .AddInterceptors(new TestQueryInterceptor (), new SampleInterceptor2());
    }
}
```

通过返回InterceptionResult<T>.SuppressWithResult()禁止执行。重要的是要注意，DbCommandInterceptor安装的其他所有组件仍将执行，并且可以通过上的HasResult属性检查其他拦截器是否已禁止执行result。

```csharp
public override InterceptionResult<object> ScalarExecuting(DbCommand command, CommandEventData eventData, InterceptionResult<object> result)
{
    if (this.ShouldSuppressExecution(command))
    {
        return InterceptionResult.SuppressWithResult<object>(null);
    }
    
    return result;
}
```

方法中引发的异常从技术上将阻止执行，不要利用这个事实，将异常用于控制流几乎总是很糟糕的设计。
你可以拦截如下清单的操作：

方法|操作
-|-
CommandCreating|在创建命令之前（注意：一切都是命令，因此它将拦截所有查询）
CommandCreated|创建命令之后但执行之前
CommandFailed[Async]|在执行过程中命令失败并出现异常后
ReaderExecuting[Async]|在执行“查询”命令之前
ReaderExecuted[Async]|执行“查询”命令后
NonQueryExecuting[Async]|在执行“非查询”命令之前（注意：非查询的一个示例是 ExecuteSqlRaw
NonQueryExecuted[Async]|执行“非查询”命令后
ScalarExecuting [Async]|在执行“标量”命令之前（注意：“标量”是存储过程的同义词）
ScalarExecuted [Async]|执行“标量”命令后
DataReaderDispose|执行命令后

这是一个耗时命令拦截

```csharp
public class MyDBCommandInterceptor: DbCommandInterceptor
{
    public static ConcurrentDictionary CommandStartTimes = new ConcurrentDictionary();
    public static ConcurrentDictionary CommandDurations = new ConcurrentDictionary();

    public override void NonQueryExecuting(DbCommand command, DbCommandInterceptionContext interceptionContext) {
        CommandStartTimes.TryAdd(command, DateTime.Now);
        base.NonQueryExecuting(command, interceptionContext);
    }

    public override void ReaderExecuting(DbCommand command, DbCommandInterceptionContext interceptionContext) {
        CommandStartTimes.TryAdd(command, DateTime.Now);
        base.ReaderExecuting(command, interceptionContext);
    }

    public override void ScalarExecuting(DbCommand command, DbCommandInterceptionContext interceptionContext) {
        CommandStartTimes.TryAdd(command, DateTime.Now);
        base.ScalarExecuting(command, interceptionContext);
    }

    public override void NonQueryExecuted(DbCommand command, DbCommandInterceptionContext interceptionContext) {
    base.NonQueryExecuted(command, interceptionContext);
    AccumulateTime(command);
    }

    public override void ReaderExecuted(DbCommand command, DbCommandInterceptionContext interceptionContext) {
    base.ReaderExecuted(command, interceptionContext);
    AccumulateTime(command);
    }

    public override void ScalarExecuted(DbCommand command, DbCommandInterceptionContext interceptionContext) {
    base.ScalarExecuted(command, interceptionContext);
    AccumulateTime(command);
    }

    private void AccumulateTime(DbCommand command) {
    if (CommandStartTimes.TryRemove(command, out
    var commandStartTime)) {
            var commandDuration = DateTime.Now - commandStartTime;
            CommandDurations.AddOrUpdate(command.CommandText, commandDuration, (_, accumulated) => commandDuration + accumulated);
        }
    }
}
```

#### EF Core 5中的Sleeper功能：调试视图
ChangeTracker.DebugView和Model.DebugView

DebugViews输出格式正确的字符串，其中有ChangeTracker的状态或模型中的元数据的信息。DebugView提供了一个漂亮的文档，您可以捕获和打印该文档，并真正了解其幕后情况。

我在调试器上花费了大量时间，以探索有关变更跟踪器了解的内容或EF Core如何解释我所描述的模型的各种详细信息。能够以这种文本格式读取此信息，甚至将其保存在文件中，因此您无需反复调试即可收集详细信息，这是EF Core 5的一项神奇功能。

在DbContext.ChangeTracker.DebugView中，您将找到ShortView和LongView属性。

例如，这里是我刚查询一个Person对象时的视图，而我的上下文仅包含一个人。

```
Person {Id: 1} Unchanged
```

这是最常用的信息-在我的上下文中，只有一个未更改的Person的ID为1。
LongView提供了有关被跟踪实体的更多详细信息。

```
Person {Id: 1} Unchanged
    Id: 1 PK
    FirstName: 'Julie'
    IsDeleted: 'False'
    LastName: 'Lerman'
    UserId: 101
    Addresses: []
```

如果要在跟踪的Person上对其进行编辑并强制上下文检测更改，则LongView除了将状态显示为Modified之外，还对LastName属性所做的更改进行记录。

```
Person {Id: 1} Modified
   Id: 1 PK
   FirstName: 'Julie'
   IsDeleted: 'False'
   LastName: 'Lermantov' Modified
   Originally 'Lerman'
   UserId: 101
   Addresses: []
```

您可以在此视图中看到一个Addresses属性。实际上，使用导航，“人”和“地址”之间存在多对多关系。EF Core在运行时推断内存中的PersonAddress实体，以便将关系数据持久化到联接表中。
当我在其“地址”集合中创建一个具有一个地址的人的图形时，您可以在ShortView中看到一个“人”，一个地址和一个推断的PersonAddress对象。长视图显示了这些对象的属性。

```
AddressPerson (Dictionary<string, object>)
    {AddressesId: 1, ResidentsId: 1} Unchanged FK
    {AddressesId: 1} FK {ResidentsId: 1}
Address {Id: 1} Unchanged
Person {Id: 1} Modified
```

我喜欢这些调试视图，这些视图可以在调试时帮助我发现被跟踪对象的状态和关系，无论我是在解决问题还是在学习它的工作方式。
让我们转到Model.DebugViews看看您可以从中学到什么。
首先，我应该阐明我的模型。使用Visual Studio中的EF Core Power Tools扩展来可视化模型。


DbContext.Model.DebugView也具有ShortView和LongView。

#### 原生Sql查询
原生sql查询使用如下两个方法进行，查询的结构只能映射到dbset关联的对象类型

```csharp
DBSet.FromSqlRaw()
DBSet.FromSqlInterpolated()
```

可以使用部分linq扩展方法

```csharp
.FromSqlRaw("select * from authors").FirstOrDefault(a=>a.Id==3)
.FromSqlRaw("select * from authors").OrderBy(a=>a.LastName)
.FromSqlRaw("select * from authors").Include(a=>a.Books)
.FromSqlRaw("select * from authors").AsNoTracking()
```

`Find`方法不受支持

#### 避免Sql注入
参数化查询

```csharp
.FromSqlRaw("select * fro mauthors where lastnmae like '{0}%'",lastnameStart).TagWith("Fromatted_Safe").ToList()
.FromSqlRaw($"select * fro mauthors where lastnmae like '{lastnameStart}%'").TagWith("Fromatted_Safe").ToList()
.FromSqlInterpolated($"select * fro mauthors where lastnmae like '{lastnameStart}%'").TagWith("Interpolated_Safe").ToList()

```

不安全的查询

```csharp
string sql = $"select * fro mauthors where lastnmae like '{lastnameStart}%'";
.FromSqlRaw(sql).TagWith("Interpolated_Unsafe").ToList()
```

```csharp
.FromSqlRaw($"select * fro mauthors where lastnmae like '{lastnameStart}%'").TagWith("Interpolated_Unsafe").ToList()
```

##### 存储过程使用

```csharp
EXEC thesproc param1,param2,param3
```

添加存储过程

```csharp
add-migration AddStoredProc
```

```csharp
public partial class AddStoredProc : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.Sql(@"
        CREATE PROCEDURE dbo.AuthorsPublishedinYearRange
        @yearstart int,
        @yearend int
        AS
        select * from authors as a
        left join books as b  on a.authorid = b.authorId
        where Year(b.PublishDate) >=@yearstart and Year(b.PublishDate) <=@yearend
        ");
    }
    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.Sql(" drop procedure AuthorsPublishedinYearRange")
    }
}
```

执行存储过程

```csharp
DBSet.FromSqlRaw("AuthorsPublishedinYearRange {0},{1}",1999,2010);
DBSet.FromSqlInterpolated($"AuthorsPublishedinYearRange {start},{end}");
```

不能使用的方法：Include

#### 视图的使用方法

```csharp
//示例，视图返回如下数据
public class AuthorByArtist
{
    public string Artist {get;set;}
    public string? Author {get;set;}
}
//1、定义Dbset
public virual DbSet<AuthorByArtist> AuthorByArtist {get;set;}
//2、设置Entity
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<AuthorByArtist>().HasNoKey()
    .ToView("AuthorByArtist");
    .....
}
```

没有主键，不能使用Find方法查询数据

#### 数据库级别执行：Non-Query Raw SQL

```CSharp
_context.Database.ExecuteSQLRaw("update author set a.name = '{0}",newname);
_context.Database.ExecuteSQLRawAsync("update author set a.name = {0}",newname);

_context.Database.ExecuteSQLInterpolated(("update author set a.name = {newname}  where authorid = {id}");
_context.Database.ExecuteSQLInterpolatedAsync("update author set a.name = {newname}  where authorid = {id}");
//执行存储过程
_context.Database.ExecuteSQLRaw("DeleteCover {0}", coverId);
```

#### Testing with EF Core
单元测试、集成测试、功能测试，其他自动测试
- 数据库测试

```csharp
[Testclass]
public class DatabaseTests
{
    [TestMethod]
    public void CanInsertAuthorIntoDatabase()
    {
        var builder = newDbContextoptionsBuilder<PubContext>();
        builder.UseSglServer("Data Source = (localdb)l\MSSQLLocalDB; Initial Catalog = PubTestData");
        using(var context = new PubContext(builder.Options))
        {
            context.Database.EnsureDeleted();
            context.Database.EnsureCreated();
            
            var author = new Author { FirstName = "a", LastName = "b" };
            context.Authors.Add(author);Debug.WriteLine($"Before save: (author.AuthorId}");
            context.SaveChanges();
            Debug.WriteLine($"After save: (author.AuthorId)");
            Assert.AreNotEqual(o, author.AuthorId);
        }
    }
}
```

asp .net core


#### Some More Practical Mappings
- 全局配置

```csharp
protected override void ConfigureConventions(ModelConfigurationBuilder configurationBuilder)
{
    //此处可配置，全部类型设置
    configurationBuilder.Properties<string>().HaveColumnType("varchar(100)");
    //全局类型转换
    configurationBuilder.Properties<BookGenre>).HaveConversion<string>();
    configurationBuilder.Properties<Color>().HaveConversion(typeof(ColorTostring))
}

//自定义转换类型
public class ColorTostring : ValueConverter<Color, string>
{
    public ColorTostring() : base(Colorstring, Colorstruct)
    {
    }
    private static Expression<Func<Color, string>> ColorString = v => new String(v.Name);
    private static Expression<Func<string, Color>> Colorstruct = v => Color.FromName(v);
}
```

#### 事务使用

```csharp
using var transaction = _dbcontext.Database.BeginTransaction();
....
_dbcontext.Database.ExecuteSqlInterpolated($"delete from books where bookid={bookid}");
_dbcontext.SaveChanges();
transcation.Commit();
....
```

并发处理

```csharp
try
{
    context.SaveChanges();
}
catch(DBUpdateConcurrencyException ex)
{
    //处理并发异常
}
```

```csharp
//使用连接池
builder.Services.AddDbContextPool<PUbContext>(opt =>{
....
opt.
    }
);

//自动重连
protected override void OnConfiguring(DbContextoptionsBuilder optionsBuilder)
{ 
    optionsBuilder.UseSglServer(myconnectionoptions=>options.EnableRetryOnFailure();
}
```

#### EF Pipeline Event
- DBContext.SaveingChanges
- DBContext.SavedChanges
- DbContext.SaveChangesFailed
- ChangeTracker.Tracked
- ChangeTracker.SateChanged

#### Intercepting Database Operations  

Interceptor|Database operations intercepted
-|-
IDbCommandInterceptor|发送命令到数据库前、命令执行失败，释放DbDataReader
IDbConnectionInterceptor|打开或关闭连接，连接失败
IDbTranscationInterceptor|创建事务，使用存在的事务，事务提交前，回滚事务，创建和使用新的保存点，事务失败

注入

```csharp
opt->opt.AddINterceptors(new MyInterceptor());
```
