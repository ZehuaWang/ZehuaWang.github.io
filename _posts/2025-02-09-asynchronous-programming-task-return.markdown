---
layout: post
title:  "Async programming return types in C#"
date:   2025-02-09 00:00:00 -0500
categories: C#
---

[Async programming task return types](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-return-types)

Async methods can have the following return types:

Task, for an asynchronous method that doesn't return a value

Task<TResult>, for an async method that returns a value

void, for an event handler

Any type that has an accessible GetAwaiter method. The object returned by the GetAwaiter method must implement the System.Runtime.CompilerServices.ICriticalNotifyCompletion interface.

IAsyncEnumerable<T> for an async method that returns an async stream.

#### Event handler

C# 中的事件处理程序（Event Handler）是用于响应特定事件的方法，通常遵循以下特征：

1. 基本结构

{% highlight csharp %}
// 典型的事件处理程序签名
void OnButtonClick(object sender, EventArgs e)
{
    // 处理点击事件的逻辑
}
{% endhighlight %}

2. 核心要素
sender：触发事件的对象
EventArgs：包含事件相关数据的参数
返回类型始终为 void

3. 使用场景

{% highlight csharp %}
// 在窗体类中订阅按钮点击事件
button.Click += OnButtonClick; 

// 自定义事件的声明
public event EventHandler<MyEventArgs> MyEvent;
{% endhighlight %}

4. 命名惯例
通常以 On 开头 + 事件名称（如 OnLoad, OnDataReceived），这与之前提到的异步方法命名规范不同，事件处理程序即使包含异步操作也不需要添加 "Async" 后缀。

5. 特殊类型

异步事件处理程序：

{% highlight csharp %}
async void OnNetworkDataReceived(object sender, DataEventArgs e)
{
    await ProcessDataAsync(e.Data); // 虽然包含异步操作，但仍保持传统命名
}
{% endhighlight %}

#### GetAwaiter

在C#异步编程中，`GetAwaiter` 是支持 `await` 操作的核心机制，属于"任务异步模式"（Task-based Asynchronous Pattern）的重要组成部分。以下是关键要点：

核心作用

任何实现了 GetAwaiter 方法的类型都可以被 await，这使得异步编程不再局限于 Task 类型。

{% highlight csharp %}
// 自定义可等待类型
public class MyAwaitable
{
    public MyAwaiter GetAwaiter() => new MyAwaiter();
}

// 使用自定义类型
await new MyAwaitable();
{% endhighlight %}

Awaiter 要求

返回的等待器（awaiter）必须满足：
- 实现 INotifyCompletion 或 ICriticalNotifyCompletion 接口
- 包含 bool IsCompleted { get; } 属性
- 提供 void GetResult() 或 TResult GetResult() 方法

编译器行为

当使用 await 时，编译器会自动生成状态机代码：

{% highlight csharp %}
var awaiter = expression.GetAwaiter();
if (!awaiter.IsCompleted)
{
    // 挂起协程，注册后续操作
    awaiter.OnCompleted(/* 续延逻辑 */);
}
return awaiter.GetResult();
{% endhighlight %}

适用场景

ValueTask：减少堆分配
Yield 操作：await Task.Yield()
自定义等待器（如 SpanAwaitable）
与 legacy 代码集成（如 WaitHandle）

