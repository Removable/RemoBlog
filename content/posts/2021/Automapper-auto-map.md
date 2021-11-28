---
title: "使用AutoMapper自动映射"
date: 2021-01-15T12:50:02+08:00
draft: false
tags: ["C#", ".NET", "AutoMapper"]
categories: ["开发记录"]
series: [""]
---

## 使用AutoMapper自动映射

#### 安装Nuget包

通过.Net CLI安装

```
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection --version 8.1.0
```

该包会自动安装AutoMapper的依赖包。

#### 配置AutoMapper

在``Startup.cs``的ConfigureServices方法中配置：

``` c#
services.AddAutoMapper(AppDomain.CurrentDomain.GetAssemblies());
//也可以如下指定项目：
//services.AddAutoMapper(AppDomain.CurrentDomain.GetAssemblies().FirstOrDefault(a => a.Location.Contains("ExampleSolution.Model")));
```

接下来在ConfigureServices中添加：

```c#
services.AddAutoMapperProfiles(Configuration);
```

然后为每个需要映射的类之间添加配置，如：

```c#
    public class ExmapleProfile : Profile
    {
        public ExmapleProfile()
        {
            //这里的映射方向是单向的，如不需要从ExmapleViewModel转Exmaple，则第二条就可以不用写
            CreateMap<Exmaple, ExmapleViewModel>();
            CreateMap<ExmapleViewModel, Exmaple>();
        }
    }
```

#### 使用

通过构造函数注入：

```c#
	private readonly IMapper _mapper;
	public ExmapleController(IMapper mapper)
    {
        _mapper = mapper;
    }
```

使用：

```c#
	public ActionResult ExmapleMethod()
    {
        ......
        Exmaple entity = new Exmaple();
        ExmapleViewModel entityVM = _mapper.Map<ExmapleViewModel>(entity);
        ......
    }
```