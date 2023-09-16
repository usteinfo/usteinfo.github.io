---
layout: fragment
title: EF Core 7.0 启用 MySQL 数据库访问
tags: [ef, mysql]
description: Entity Framwork Core 中启用MySQL数据库访问。
keywords: ef, mysql
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

Entity Framwork Core 7.0 中启用MySQL数据库访问。

1、引用包

```xml
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="7.0.7" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="7.0.7">
<PackageReference Include="Pomelo.EntityFrameworkCore.MySql" Version="7.0.0" />
```

2、安装ef工具

```shell
dotnet tool install -g dotnet-ef
dotnet tool update -g dotnet-ef
```

3、建立dbcontext

```csharp
namespace WebApi.Helpers;

using Microsoft.EntityFrameworkCore;
using WebApi.Entities;

public class DataContext : DbContext
{
    protected readonly IConfiguration Configuration;

    public DataContext(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        // connect to mysql with connection string from app settings
        var connectionString = Configuration.GetConnectionString("WebApiDatabase");
        options.UseMySql(connectionString, ServerVersion.AutoDetect(connectionString));
    }

    public DbSet<User> Users { get; set; }
}
```

4、配置连接字符串

```json
{
    "ConnectionStrings": {
        "WebApiDatabase": "server=localhost; database=dotnet-5-crud-api; user=testUser; password=testPass123"
    },
    "Logging": {
        "LogLevel": {
            "Default": "Information",
            "Microsoft.AspNetCore": "Warning"
        }
    }
}
```

5、创建数据库脚本

```shell
dotnet ef migrations add InitialCreate
donet ef migrations remove
dotnet ef database update
```


