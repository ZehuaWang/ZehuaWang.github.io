---
layout: post
title:  "Sporch Logging"
date:   2025-02-15 00:00:00 -0500
categories: work
---

#### ILoggingBuilder

ILoggingBuilder 是 .NET Core/5/6+ 中用于配置日志系统的重要接口，属于 Microsoft.Extensions.Logging 命名空间。以下是关键要点：

{% highlight csharp %}
// 典型使用场景（Program.cs）
var builder = WebApplication.CreateBuilder(args);

// 通过 ILoggingBuilder 配置日志
builder.Logging.AddConsole()  // 添加控制台日志
        .AddDebug()           // 添加调试输出
        .SetMinimumLevel(LogLevel.Information); // 设置最低日志级别
{% endhighlight %}

主要功能：

- 添加日志提供程序（Console/Debug/EventSource 等）
- 配置日志过滤规则
- 设置默认日志级别
- 集成第三方日志系统（Serilog/NLog 等）

常用方法：

{% highlight csharp %}
// 添加日志提供程序
services.AddLogging(logging => 
{
    logging.AddAzureWebAppDiagnostics(); // Azure 诊断
    logging.AddEventLog();              // Windows 事件日志
});

// 配置过滤规则
logging.AddFilter("Microsoft", LogLevel.Warning) // 过滤 Microsoft 命名空间的日志
       .AddFilter<DebugLoggerProvider>("MyApp", LogLevel.Debug); // 特定提供程序的过滤
{% endhighlight %}

自定义日志配置示例：

{% highlight csharp %}
// 创建自定义日志提供程序
class MyCustomLoggerProvider : ILoggerProvider
{
    public ILogger CreateLogger(string categoryName) => new MyCustomLogger();
    public void Dispose() { }
}

// 通过 ILoggingBuilder 注册
builder.Logging.AddProvider(new MyCustomLoggerProvider());
{% endhighlight %}

与 Azure 集成示例：

{% highlight csharp %}
// 在 Azure Functions 中配置 Application Insights
builder.Logging.AddApplicationInsights(
    configureTelemetryConfiguration: config => 
        config.ConnectionString = "YOUR_CONNECTION_STRING",
    configureApplicationInsightsLoggerOptions: options => { }
);{% endhighlight %}

最佳实践：

在 Program.cs 或 Startup.ConfigureServices 中配置

使用 appsettings.json 进行动态配置：

{% highlight json %}
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    },
    "Console": {
      "LogLevel": {
        "MyApp.Controllers": "Debug"
      }
    }
  }
}
{% endhighlight %}

#### BaseProcessor under OpenTelemetry namespce

BaseProcessor应该是OpenTelemetry SDK中的一个组件，用于处理遥测数据。记得OpenTelemetry中的Processor用于处理导出前的数据，比如过滤、修改或批量处理。BaseProcessor可能是一个抽象基类，供用户自定义处理器时继承。

核心作用:

{% highlight csharp %}
// 继承关系
public abstract class BaseProcessor<T> : IDisposable
    where T : class    
{% endhighlight %}

主要功能：

数据过滤：在导出前过滤不需要的 spans

数据增强：添加自定义属性到 spans

批处理：优化导出性能

错误处理：自定义导出失败处理逻辑

数据转换：修改或标准化数据格式

典型实现示例：

{% highlight csharp %}
using OpenTelemetry;

// 自定义 Processor
public class CustomProcessor : BaseProcessor<Activity>
{
    public override void OnStart(Activity data)
    {
        // 添加自定义标签
        data.SetTag("deployment.environment", "production");
    }

    public override void OnEnd(Activity data)
    {
        // 过滤掉持续时间小于 100ms 的 span
        if (data.Duration.TotalMilliseconds < 100)
        {
            data.ActivityTraceFlags |= ActivityTraceFlags.Recorded;
        }
    }
}

// 注册 Processor
var tracerProvider = Sdk.CreateTracerProviderBuilder()
    .AddSource("MyApp")
    .AddProcessor(new CustomProcessor())
    .Build();
{% endhighlight %}

#### GenevaLogExporter 是 OpenTelemetry 中用于将日志数据导出到 Microsoft Geneva（现 Azure Monitor）的专用组件，属于 OpenTelemetry.Exporter.Geneva 命名空间。

{% highlight csharp %}
// 典型配置示例（Program.cs）
using OpenTelemetry.Logs;
using OpenTelemetry.Exporter.Geneva;

var builder = WebApplication.CreateBuilder(args);

builder.Logging.AddOpenTelemetry(options => 
{
    options.AddGenevaLogExporter(exporterOptions => 
    {
        exporterOptions.ConnectionString = "Account=...;Namespace=...";
        exporterOptions.PreferredProtocol = GenevaProtocol.Etw;
        exporterOptions.CustomFields = new[] { "userId", "requestId" };
    });
});
{% endhighlight %}

主要功能:

协议支持：

ETW (Event Tracing for Windows)：高性能 Windows 原生协议

Unix Domain Socket：跨平台支持（Linux/Windows）

TCP：传统网络传输

数据增强：

{% highlight csharp %}
// 添加自定义元数据
exporterOptions.TableNameMappings = new Dictionary<string, string>
{
    ["*"] = "MyAppLogs" // 所有日志使用相同表
};
{% endhighlight %}

性能优化：

批量导出（默认每 2 秒或 1000 条）

内存缓冲区（可配置队列大小）

压缩传输（支持 GZip）

与 Azure Monitor 集成

{% highlight csharp %}
// 同时导出到 Application Insights 和 Geneva
builder.Logging.AddOpenTelemetry(options =>
{
    options.AddGenevaLogExporter(genevaOptions => 
    {
        genevaOptions.ConnectionString = "InstrumentationKey=...";
    });
    
    options.AddAzureMonitorLogExporter(azureOptions => 
    {
        azureOptions.ConnectionString = "InstrumentationKey=...";
    });
});
{% endhighlight %}

高级配置示例

{% highlight csharp %}
var loggerFactory = LoggerFactory.Create(builder =>
{
    builder.AddOpenTelemetry(options =>
    {
        options.IncludeScopes = true;
        options.ParseStateValues = true;
        
        options.AddGenevaLogExporter(o => 
        {
            o.ConnectionString = "Account=myaccount;Namespace=mynamespace";
            o.BatchSize = 500;
            o.MaxQueueSize = 5000;
            o.Protocol = GenevaProtocol.Tcp;
            o.TcpPort = 1234;
            o.ExporterTimeoutMilliseconds = 30000;
        });
    });
});
{% endhighlight %}

数据映射规则：

自动转换 OpenTelemetry 字段到 Geneva 模式

TraceId → OperationId

SpanId → RequestId

LogLevel → SeverityLevel

混合云支持：

支持本地 Geneva 部署

兼容 Azure 公有云和 Azure Stack

故障排查

{% highlight bash %}
# 检查 ETW 会话（Windows）
logman query -ets

# 监控 Unix Domain Socket（Linux）
sudo lsof -U | grep geneva

# 查看导出统计
dotnet-counters monitor --counters OpenTelemetry.Exporter.Geneva{% endhighlight %}

#### <span style="color:green">ILogger</span>

核心作用

{% highlight csharp %}
// 接口定义
public interface ILogger
{
    void Log<TState>(
        LogLevel logLevel,
        EventId eventId,
        TState state,
        Exception? exception,
        Func<TState, Exception?, string> formatter);
    
    bool IsEnabled(LogLevel logLevel);
    IDisposable? BeginScope<TState>(TState state);
}
{% endhighlight %}

典型使用场景

基础日志记录：

{% highlight csharp %}
public class OrderController : Controller
{
    private readonly ILogger<OrderController> _logger;

    public OrderController(ILogger<OrderController> logger)
    {
        _logger = logger;
    }

    public IActionResult Create(Order order)
    {
        _logger.LogInformation("Creating order {OrderId}", order.Id);
        // ...
    }
}
{% endhighlight %}

结构化日志（与 Azure 集成）：

{% highlight csharp %}
_logger.LogWarning("Low inventory for {ProductId} in {Region}", 
    productId, 
    region,
    new Dictionary<string, object>
    {
        ["CurrentStock"] = 5,
        ["Threshold"] = 10
    });
{% endhighlight %}

关键功能

日志级别控制：

{% highlight csharp %}
if (_logger.IsEnabled(LogLevel.Debug))
{
    _logger.LogDebug("Processing details: {Details}", GetExpensiveDetails());
}
{% endhighlight %}

作用域管理：

{% highlight csharp %}
using (_logger.BeginScope(new { OrderId = order.Id, User = user.Name }))
{
    _logger.LogInformation("Processing payment");
    // 所有在此作用域内的日志都会自动携带 OrderId 和 User
}
{% endhighlight %}

异常日志记录：

{% highlight csharp %}
try
{
    // ...
}
catch (Exception ex)
{
    _logger.LogError(ex, "Failed to process order {OrderId}", orderId);
}
{% endhighlight %}

与 OpenTelemetry 集成

{% highlight csharp %}
// 配置 OpenTelemetry 日志收集
var loggerFactory = LoggerFactory.Create(builder =>
{
    builder.AddOpenTelemetry(options =>
    {
        options.AddConsoleExporter();
        options.AddAzureMonitorLogExporter();
    });
});

ILogger logger = loggerFactory.CreateLogger<Program>();
logger.LogInformation("App started with {Args}", args);
{% endhighlight %}

最佳实践

结构化日志模板

{% highlight csharp %}
// ✅ 正确：使用命名占位符
_logger.LogInformation("User {UserId} logged in from {IP}", userId, ip);

// ❌ 避免：字符串拼接
_logger.LogInformation($"User {userId} logged in from {ip}");
{% endhighlight %}

性能优化：

{% highlight csharp %}
// 使用 LoggerMessage 模式（高性能）
private static readonly Action<ILogger, int, Exception?> _orderProcessed = 
    LoggerMessage.Define<int>(
        LogLevel.Information,
        new EventId(1001, "OrderProcessed"),
        "Order {OrderId} processed");

_orderProcessed(_logger, orderId, null);
{% endhighlight %}

常用日志级别快捷方法：

{% highlight csharp %}
_logger.LogTrace("Verbose details");
_logger.LogDebug("Debug information");
_logger.LogInformation("Order {Id} created", orderId);
_logger.LogWarning("Unexpected value {Value}", input);
_logger.LogError(ex, "Operation failed");
_logger.LogCritical("System shutdown!");
{% endhighlight %}