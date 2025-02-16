---
layout: post
title:  "Interface in C#"
date:   2025-02-14 00:00:00 -0500
categories: C#
---

接口（Interface）核心概念

定义与作用：

接口定义一组相关功能的契约，要求非抽象类或结构体必须实现这些功能。

支持多继承行为（弥补 C# 不支持类多继承的限制）。

成员类型：

静态方法：必须包含实现（C# 8+）。

默认方法：可为成员提供默认实现（C# 8+）。

抽象成员：无默认实现，需由实现类/结构体具体定义。

禁止声明：实例字段、自动属性、属性风格事件。

适用场景：

为类添加多来源行为（如 类A 实现 接口X 和 接口Y）

为结构体（struct）模拟继承（因结构体不支持基类继承）

示例说明

1. 接口定义

{% highlight csharp %}
public interface ILogger
{
    // 抽象方法（需实现）
    void Log(string message);

    // 默认方法（可选重写）
    public string FormatMessage(string message) => $"[LOG] {message}";
}
{% endhighlight %}

2. 类实现接口

{% highlight csharp %}
public class FileLogger : ILogger
{
    // 必须实现 Log 方法
    public void Log(string message)
    {
        File.WriteAllText("log.txt", message);
    }

    // 可选择重写默认方法
    public string FormatMessage(string message) => $"[FILE] {message}";
}
{% endhighlight %}

3. 结构体实现接口

{% highlight csharp %}
public struct ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
    // 使用默认的 FormatMessage 方法
}
{% endhighlight %}

关键优势

多态支持：通过接口实现松耦合设计。

代码复用：通过默认方法减少重复代码。

跨类型约束：统一不同类/结构体的行为标准。

总结

接口是 C# 中实现多继承行为和类型约束的核心机制，尤其适用于需要跨类/结构体统一功能定义的场景。