---
title: "AspectCore 配合 Log4Net 全局记录Asp.Net Core Web Api异常日志"
date: 2021-02-27T12:50:02+08:00
draft: false
tags: ["C#", ".NET", "Log"]
categories: ["开发记录"]
series: [""]
---

### AspectCore 配合 Log4Net 全局记录Asp.Net Core Web Api异常日志

#### 配置AspectCore-Framework

首先通过Nuget安装`AspectCore.Extensions.DependencyInjection`。

接下来我们可以**配置拦截器**：

```c#
    //继承自AbstractInterceptorAttribute
    public class CustomInterceptorAttribute : AbstractInterceptorAttribute
    {
        public override async Task Invoke(AspectContext context, AspectDelegate next)
        {
            try
            {
                Console.WriteLine("调用前");
                await next(context);//执行调用的方法等
            }
            catch (Exception ex)
            {
                Console.WriteLine("捕获到异常");
                throw;
            }
            finally
            {
                Console.WriteLine("调用结束");
            }
        }
    }
```

然后在`Startup.cs`中的`ConfigureServices`方法中**配置代理**

```c#
    services.(config =>
    {
        services.AddTransient<IExmapleService, ExmapleService>();
        
        services.AddControllers();
        
        config.Interceptors.AddTyped<CustomInterceptorAttribute>(Predicates.ForService("*Service"));
        config.Interceptors.AddTyped<CustomInterceptorAttribute>(Predicates.ForService("*Controller"));
    });
```

以上配置是对名称以Service或Controller结尾的进行代理，还有其他一些规则如：

```c#
	//全部代理
	config.Interceptors.AddTyped<CustomInterceptorAttribute>();
	//名称以Execute开头的方法会被代理
	config.Interceptors.AddTyped<CustomInterceptorAttribute>(Predicates.ForMethod("Execute*"));
	//以Service结尾则不会被代理
	config.NonAspectPredicates.AddService("Service");
	//更多详细规则可以查阅官方文档
	//https://github.com/dotnetcore/AspectCore-Framework/blob/master/docs/1.%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97.md
	......
```

也可以通过`NonAspectAttribute`对Service或Method进行单独设置：

```c#
	//无论在Startup.cs中如何配置，该接口都不会通过代理
	[NonAspect]
	public interface IExampleService
	{
    	void Method();
	}
```

最后在``Program.cs``中，将依赖注入交由AspectCore处理：

```c#
	public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                })
        		//略
                .UseServiceProviderFactory(new DynamicProxyServiceProviderFactory());
```

一般来说到这里就完成了，但是我在上面配置代理时，将Controller也配置为进行代理：``config.Interceptors.AddTyped<CustomInterceptorAttribute>(Predicates.ForService("*Controller"));``，因此，就需要进行一些额外的处理。

- 首先是在``Startup.cs``中，将Controller作为Service注册：

  ```c#
  services.AddControllers().AddControllersAsServices(); //MVC的话AddControllers就是AddMvc()
  ```

- 然后将action都标记为虚函数，如：

  ```c#
  	public class ExampleController
      {
          [HttpPost]
          public virtual ActionResult Test()
          {
              //......
          }
      }
  ```

这样就可以在Controller层进行拦截了。



#### 配置Log4Net

依旧是通过Nuget安装包``Microsoft.Extensions.Logging.Log4Net.AspNetCore``

在``Startup.cs``中**对Configure进行一些修改**：

```c#
	//这里多加一个参数
	public void Configure(IApplicationBuilder app, IWebHostEnvironment env, ILoggerFactory loggerFactory)
    {
        ...... //其他代码

        loggerFactory.AddLog4Net();
    }
```

然后添加**配置文件``log4net.config``**

```xml
<?xml version="1.0" encoding="utf-8"?>

<log4net>
    <!-- 日志记录的相关配置，可以针对不同记录方式和级别，配置多个appender节点 -->
    <appender name="ErrorLog" type="log4net.Appender.RollingFileAppender">
        <!-- 日志文件保存位置，可以相对路径也可以绝对路径，注意这里是斜杠而不是反斜杠 -->
        <file value="logs/" />
        <!-- 文件的命名规则，此处为日期加.log后缀 -->
        <datePattern value="yyyy-MM-dd'.log'" />
        <appendToFile value="true" />
        <rollingStyle value="Size" />
        <maxSizeRollBackups value="100" />
        <maximumFileSize value="3MB" />
        <!-- 按日期保存 -->
        <rollingStyle value="Date" />
        <!-- 静态文件名设为false -->
        <staticLogFileName value="false" />
        <!-- 日志文件最小锁定模式，尽量保证更多的同时写入 -->
        <lockingModel type="log4net.Appender.FileAppender+MinimalLock" />
        <layout type="log4net.Layout.PatternLayout">
            <conversionPattern
                value="%newline-----------------------%newline【时间】：%d%newline【级别】：%-5p%newline【对象】：%logger%newline【内容】：%m%newline" />
        </layout>
        <filter type="log4net.Filter.LevelRangeFilter">
            <!--设置该日志文件将记录的日志级别区间，如果希望每个级别都分开一个文件记录，可以将appender节点配置多个，设置不同的记录级别-->
            <param name="LevelMin" value="Error" />
            <param name="LevelMax" value="Error" />
        </filter>
    </appender>

    <root>
        <!--注意，当配置多个不同日志级别的appender节点时，务必按以下级别顺序排列appender-ref节点，否则低级别将不会执行-->
        <!--   All -> Debug -> Info -> Warn -> Error -> Fatal -> Off   -->
        <appender-ref ref="ErrorLog" />
        <level value="DEBUG" />
    </root>
</log4net>
```

最后再修改一下文章开头的拦截器部分，记录日志：

```c#
    public class CustomInterceptorAttribute : AbstractInterceptorAttribute
    {	
     	//在这里注入日志服务，其他所需服务也可以这样注入
        [FromServiceContext] private ILogger<CustomInterceptorAttribute> Logger { get; set; }
        
        public override async Task Invoke(AspectContext context, AspectDelegate next)
        {
            try
            {
                Console.WriteLine("调用前");
                await next(context);//执行调用的方法等
            }
            catch (Exception ex)
            {
                Console.WriteLine("捕获到异常");
                //记录日志，这里简单的ToString了，具体记录内容自行调整
                Logger.LogError(ex.ToString());
                //记录日志后是继续抛出还是进行其他处理，可自行调整
                throw;
            }
            finally
            {
                Console.WriteLine("调用结束");
            }
        }
    }
```



到这里应该就可以在Service或Controller中发生异常时进行记录了。其实通过AspectCore的这一功能，可以实现更多的功能，记录日志只是其中之一。