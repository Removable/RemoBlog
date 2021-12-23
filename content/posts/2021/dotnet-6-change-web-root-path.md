---
title: "ASP.NET 6 修改 WebRoot 路径"
date: 2021-12-23T15:29:21+08:00
draft: false
tags: ["C#", "Asp.Net Core", ".Net"]
categories: ["C#代码", "开发记录"]
series: [""]
---

## 问题
最近的项目中需要修改 Web Root 路径，按照老方法发现报异常，于是 Google 得知相关方法在 ASP.NET 6 中有所修改。

## 代码

### ASP.NET 5

```c#
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            //这里指定新位置
            webBuilder.UseWebRoot("webroot")
                      .UseStartup<Startup>();
        });
```

### ASP.NET 6

```c#
var builder = WebApplication.CreateBuilder(new WebApplicationOptions
{
    Args = args,
    //这里指定新位置，也可以使用绝对路径
    WebRootPath = "webroot"
});

var app = builder.Build();
```

> 若要修改 Content Root 路径也可以在这里一并修改。

## 总结

根据[微软官方文档](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/?view=aspnetcore-6.0&tabs=windows#web-root)解释：

> Content 根目录是指向以下内容的基路径：
>
> - 托管应用的可执行文件 (.exe)。
>
> - 构成应用程序的已编译程序集 (.dll)。
>
> - 应用使用的内容文件，例如：
>   - Razor 文件（.cshtml、.razor）
>   - 配置文件（.json、.xml）
>   - 数据文件 (.db)
>   
> - [Web 根目录](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/?view=aspnetcore-6.0&tabs=windows#web-root)，通常是 wwwroot 文件夹。
>
> 在开发中，内容根目录默认为项目的根目录。 此目录还是应用内容文件和 Web 根目录的基路径。 在构建主机时设置路径，可指定不同的内容根目录。

> Web 根目录是公用静态资源文件的基路径，例如：
>
> - 样式表 (.css)
> - JavaScript (.js)
> - 图像（.png、.jpg）