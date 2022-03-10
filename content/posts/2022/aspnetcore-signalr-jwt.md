---
title: "Asp.Net Core 6 与 SignalR 使用 JWT 身份验证"
date: 2022-03-09T21:57:47+08:00
draft: false
tags: ["SignalR", "Asp.Net Core", "TypeScript", "Jwt"]
categories: ["开发记录"]
series: [""]
---

## JWT 是什么？

这一点网上有很多的描述了，在此就不多赘述。总之，JWT 的 Token 通常由三部分组成：Header，Payload 和 Signature.

Header 中包含了该 Token 所使用的签名算法；Payload 中则含有一些需要传递的信息；Signature 则是对 Header 和 Payload 的签名，用以校验前两部分有没有被篡改。

## 在 Asp.Net Core Web Api 中集成  JWT

### 安装 Nuget 包

首选我们需要安装三个 Nuget 包，分别是：

`Microsoft.AspNetCore.Authentication.JwtBearer`

`using Microsoft.IdentityModel.Tokens;`

`using System.IdentityModel.Tokens.Jwt;`

### JWT 相关参数配置

在`appsettings.json`文件中，我们添加以下配置：

```json
"JwtConfiguration": {
  "AccessSecret": "123456",
  "RefreshSecret": "654321",
  "Issuer": "this",
  "Audience": "that",
  "AccessExpiration": 3600,
  "RefreshExpiration": 2592000,
  "ClockSkew": 60
}
```

以上配置中，具体字段名和内容可以自行修改。

然后我们新建一个类，用于反序列化配置内容：

```c#
/// <summary>
/// JWT Token配置
/// </summary>
public class JwtConfiguration
{
    /// <summary>
    ///     AccessToken密钥
    /// </summary>
    public string AccessSecret { get; set; } = string.Empty;
    /// <summary>
    ///     RefreshToken密钥
    /// </summary>
    public string RefreshSecret { get; set; } = string.Empty;
    /// <summary>
    ///     签发人
    /// </summary>
    public string Issuer { get; set; } = string.Empty;
    /// <summary>
    ///     受众
    /// </summary>
    public string Audience { get; set; } = string.Empty;
    /// <summary>
    ///     AccessToken有效时长
    /// </summary>
    public int AccessExpiration { get; set; }
    /// <summary>
    ///     RefreshToken有效时长
    /// </summary>
    public int RefreshExpiration { get; set; }
    /// <summary>
    ///     允许的时差
    /// </summary>
    public int ClockSkew { get; set; }
}
```

如果配置文件中的字段名修改了，那么这里的字段名也要相应修改。

### 在 Program.cs 中配置 JWT

为避免`Program.cs`文件太臃肿，我们可以单独新建一个文件，并添加一个`WebApplicationBuilder`的扩展方法：

```C#
/// <summary>
/// JWT配置
/// </summary>
public static class JwtSetup
{
    /// <summary>
    /// JWT配置
    /// </summary>
    public static void AddJwtBearer(this WebApplicationBuilder builder)
    {
        //将配置文件中的相关内容反序列化
        var tokenInfo = builder.Configuration.GetSection("JwtConfiguration").Get<JwtConfiguration>();
        builder.Services.AddAuthentication(x =>
        {
            x.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
            x.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
        }).AddJwtBearer(x =>
        {
            x.Events = new JwtBearerEvents
            {
                //验证失败时的处理
                OnAuthenticationFailed = context =>
                {
                    //若失败类型为过期，则返回特定Header，便于客户端判断
                    if (context.Exception.GetType() == typeof(SecurityTokenExpiredException))
                        context.Response.Headers.Add("tokenErr", "expired");
                    return Task.CompletedTask;
                },
            };
            x.RequireHttpsMetadata = false;
            x.SaveToken = true;
            x.TokenValidationParameters = new TokenValidationParameters
            {
                ValidateIssuerSigningKey = true,
                IssuerSigningKey = new SymmetricSecurityKey(Encoding.ASCII.GetBytes(tokenInfo.AccessSecret)),
                ValidAlgorithms = new[] { SecurityAlgorithms.HmacSha256 },
                ValidIssuer = tokenInfo.Issuer,
                ValidAudience = tokenInfo.Audience,
                ValidateIssuer = true,
                ValidateAudience = true,
                ValidateLifetime = true,
                ClockSkew = TimeSpan.FromSeconds(tokenInfo.ClockSkew)
            };
        });
    }
}
```

这样一来，我们只需在`Program.cs`中加入一行就可以了：

```c#
builder.AddJwtBearer();
```

另外，我们还需要加入另外一行，以便添加相应的身份验证中间件：

```c#
app.UseAuthentication();
```

需要注意的是，文件中可能还有一行添加权限验证中间件的 `app.UseAuthorization()` 与之非常相似，我们要添加的 `app.UseAuthentication()` 一定要加在它的上方，这个顺序不能乱。

### 生成 Token

接下来，我们添加一个创建 Access Token 的方法：

```C#
private string GenerateAccessToken(string username)
{
    var claims = new[]
    {
        new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
        new Claim("username", username),
        //也可以添加其他 Claim
    };
    var key = new SymmetricSecurityKey(Encoding.ASCII.GetBytes(tokenInfo.AccessSecret));
    var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
    var jwtAccessToken = new JwtSecurityToken(tokenInfo.Issuer, tokenInfo.Audience, claims,
        expires: DateTime.Now.AddSeconds(tokenInfo.AccessExpiration), signingCredentials: credentials,
        notBefore: DateTime.Now);
    return new JwtSecurityTokenHandler().WriteToken(jwtAccessToken);
}
```

同理，我们还可以添加一个创建 Refresh Token 的方法，用于在 Access Token 过期时进行刷新操作：只需将签名密钥和过期时长换成相应的配置即可。

### 设置需要验证或者接受匿名的 Controller 或 Action

现在，我们只需在要进行身份验证的 Controller 或者 Action 上添加相应的 Attribute 即可，如：

```c#
//可以添加在这里，即整个Controller都需要身份验证才可访问
[Authorize]
public class ExampleController : Controller
{
	//或者添加在这里，即只有该方法需要验证
	[Authorize]
	public void TestAuthorize()
	{
		//...
	}
    //这个Attribute表示该方法接受匿名访问，无需身份验证，也可以添加在Controller上。改标签优先于[Authorize]，即Controller上标记[Authorize]时，若Action有[AllowAnonymous]，则该Action也可匿名访问
    [AllowAnonymous]
    public void TestAnonymous()
    {
        //...
    }
}
```

我们可以启动项目，调用添加了`[Authorize]`的方法试一试，这时应该会返回401的错误，表示接口需要进行验证。而添加`[AllowAnonymous]`的方法可以正常访问。

### 客户端获取及使用 Token

那么客户端在调用需要身份验证的接口之前，需要调用一个可匿名访问的登录接口。登录接口在验证了客户端的身份以后，就可以调用相应方法生成 Access Token 以及 Refresh Token 并返回给客户端。

客户端在拿到 Access Token 后，只需在请求的 Headers 中添加：

```
"Authorization": "Bearer {获取的Token}"
```

注意：Bearer 和 Token之间是有一个空格的。

### 刷新 Token

通常，Access Token 的有效期都比较短，这样的话，客户端就需要频繁地重新登录，这种体验当然是非常不好的。而设置过长的 Access Token 有效期，又大大降低了安全性。因此，前面咱们提到了生成并返回 Refresh Token，而 Refresh Token 的有效期很长（如7天或一个月），客户端在发现 Access Token 过期后，可以调用相应接口，并传入 Access Token 和 Refresh Token来刷新令牌：

```c#
public (bool result, string msg) RefreshToken(string refreshToken, string accessToken)
{
    var handle = new JwtSecurityTokenHandler();
    //先验证RefreshToken
    try
    {
        handle.ValidateToken(refreshToken, new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidIssuer = tokenInfo.Issuer,
            ValidAudience = tokenInfo.Audience,
            IssuerSigningKey =
                new SymmetricSecurityKey(Encoding.ASCII.GetBytes(tokenInfo.RefreshSecret)),
            ValidateLifetime = true,
            RequireExpirationTime = true
        }, out _);
    }
    catch (Exception e)
    {
        Console.WriteLine(e.Message);
        return (false, "无效的RefreshToken");
    }
    //再验证AccessToken是否为已过期状态，若为无效Token则不予刷新
    try
    {
        handle.ValidateToken(accessToken, new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidIssuer = tokenInfo.Issuer,
            ValidAudience = tokenInfo.Audience,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.ASCII.GetBytes(tokenInfo.AccessSecret)),
            ValidateLifetime = true,
            RequireExpirationTime = true
        }, out _);
    }
    //当状态为已过期时继续
    catch (SecurityTokenExpiredException)
    {
    }
    //其他异常视为无效Token
    catch (Exception e)
    {
        Console.WriteLine(e.Message);
        return (false, "无效的AccessToken");
    }
    {
        //解析AccessToken信息
        var tokenInfo = handle.ReadJwtToken(accessToken);
        if (tokenInfo == null) return (false, "Token解析错误");
        var newAccessToken = GenerateAccessToken(tokenInfo.Payload["username"].ToString());
        return string.IsNullOrWhiteSpace(newAccessToken) ? (false, "") : (true, newAccessToken);
    }
}
```

当 Refresh Token 有效，且 Access Token 仅有过期错误时，为客户端重新生成新的 Access Token。Refresh Token 仅在调用刷新接口时作为参数传送，降低了泄露风险。

## 在 SignalR 中集成  JWT

Asp.Net Core 中如何配置 SignalR 服务端就不在本文的范围之内了，具体可以看相关教程。

我们以 TypeScript 为例。安装并引用`@microsoft/signalr`包后，新建连接实例：

```typescript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("https://****/hub", {
        accessTokenFactory(): string | Promise<string> {
            let token = sessionStorage.getItem("accessToken");//在此之前需调用登录接口获取并设置AccessToken
            return token ?? "";
        },
        httpClient: new CustomHttpClient() //这里的CustomHttpClient见下方
    })
    .withAutomaticReconnect()
    .build();
```

为实现刷新 Access Token 的需求，我们还需替换 SignalR 默认的 httpClient，并调用刷新接口：

```typescript
const getAuthHeaders = () => {
    return {
        Authorization: `Bearer ${sessionStorage.getItem("accessToken")}`,
    };
};

//调用刷新令牌的接口
const refresh = async () => {
    await fetch("https://****/Controller/RefreshToken", {
        method: "post",
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            RefreshToken: sessionStorage.getItem("refreshToken"),
            AccessToken: sessionStorage.getItem("accessToken")
        })
    }).then(async res => {
        let data = await res.json()
        sessionStorage.setItem("data["accessToken"]", data["accessToken"]);
    }).catch(err => {
        console.log(err)
    })
}

class CustomHttpClient extends signalR.DefaultHttpClient {
    constructor() {
        super(console);
    }

    public async send(
        request: signalR.HttpRequest
    ): Promise<signalR.HttpResponse> {
        const authHeaders = getAuthHeaders();
        request.headers = {...request.headers, ...authHeaders};

        await super.send(request).then(res => {
            return res;
        }).catch(async (err) => {
            if (err instanceof signalR.HttpError) {
                const error = err as signalR.HttpError;
                if (error.statusCode === 401) {
                    //这里似乎无法获取响应头，也就没法拿到之前设置的自定义tokenErr头。目前还没找到解决方案，只能默认401即为过期，刷新令牌
                    await refresh().then(r => {
                        const authHeaders = getAuthHeaders();
                        request.headers = {...request.headers, ...authHeaders};
                    });
                }
            } else {
                throw err;
            }
        });
        //重新发送一次请求
        return super.send(request);
    }
}
```

还记得之前我们在 Asp.Net Core 中配置 JWT 时添加的 `OnAuthenticationFailed` 事件吗？我们接着在它的下方添加一个新的事件：

```c#
x.Events = new JwtBearerEvents
{
    //验证失败时的处理
	OnAuthenticationFailed = context =>
	{
    	//若失败类型为过期，则返回特定Header，便于客户端判断
    	if (context.Exception.GetType() == typeof(SecurityTokenExpiredException))
        	context.Response.Headers.Add("tokenErr", "expired");
    	return Task.CompletedTask;
	},
    //添加接收消息时的事件
    OnMessageReceived = context =>
    {
        var accessToken = context.Request.Query["access_token"];
        var path = context.HttpContext.Request.Path;
        if (!string.IsNullOrWhiteSpace(accessToken) &&
            path.StartsWithSegments("/hub"))//这里可以修改为你相应的hub地址
        {
            context.Token = accessToken;
        }
        return Task.CompletedTask;
    },
};
```

好了，到这里就完成了，可以试试看 SignalR 连接在收到401错误后，会不会自动刷新令牌并重新发送请求吧。
