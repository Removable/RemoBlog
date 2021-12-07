---
title: "Asp.Net Core使用Quartz.NET实现简单定时任务"
date: 2021-12-07T14:12:02+08:00
draft: false
tags: ["C#", ".NET", "Quartz.NET"]
categories: ["开发记录"]
series: [""]
---

## 所需 Nuget 包

- Quartz.AspNetCore
- Quartz
- Quartz.Extensions.DependencyInjection
- Quartz.Extensions.Hosting

本示例仅需从nuget安装第一个包即可，其他三个包会通过依赖关系自动安装。



##  示例代码

本文基于.NET 6最新的模板。




新建一个cs文件，如`ExampleJob.cs`：

```c#
[DisallowConcurrentExecution]
public class ExampleJob : IJob
{
    public async Task Execute(IJobExecutionContext context)
    {
        try
        {
            //这里是需要定时执行的相关代码
        }
        catch (Exception e)
        {
            //异常处理
        }
    }
}
```



在`Program.cs`中添加:

```c#
builder.Services.AddQuartz(config =>
{
    config.UseDefaultThreadPool(options => { options.MaxConcurrency = 2; });
    config.UseMicrosoftDependencyInjectionJobFactory();
    config.ScheduleJob<ExampleJob>(trigger => trigger
        .WithIdentity("ExampleTrigger")
        //立即开始第一次执行
        .StartNow()
        //此后每次执行的间隔，这里是1小时，并且一直重复下去
        .WithSimpleSchedule(x=>x.WithIntervalInHours(1).RepeatForever())
        .WithDescription("A simple example"));
});

// Quartz.Extensions.Hosting hosting
builder.Services.AddQuartzHostedService(options =>
{
    options.WaitForJobsToComplete = true;
});
```

到这里，一个简单的定时任务就已经配置完毕了。项目运行起来以后，Quartz.NET将会根据以上设置定时执行任务。