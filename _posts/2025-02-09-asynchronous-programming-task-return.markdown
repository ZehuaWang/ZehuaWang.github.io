---
layout: post
title:  "Async programming return types in C#"
date:   2025-02-09 00:00:00 -0500
categories: C#
---

[Async programming task return types](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-return-types)

Async methods can have the following return types:

Task, for an asynchronous method that doesn't return a value

Task\<TResult\>, for an async method that returns a value

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


#### Task return type

Async methods that don't contain a return statement or that contain a return statement that doesn't return an operand usually have a return type of Task. Such methods return void if they run synchronously. If you use a Task return type for an async method, a calling method can use an await operator to suspend the caller's completion until the called async method finishes.

{% highlight csharp %}
public static async Task DisplayCurrentInfoAsync()
{
    await WaitAndApologizeAsync();

    Console.WriteLine($"Today is {DateTime.Now:D}");
    Console.WriteLine($"The current time is {DateTime.Now.TimeOfDay:t}");
    Console.WriteLine("The current temperature is 76 degrees.");
}

static async Task WaitAndApologizeAsync()
{
    await Task.Delay(2000);

    Console.WriteLine("Sorry for the delay...\n");
}
// Example output:
//    Sorry for the delay...
//
// Today is Monday, August 17, 2020
// The current time is 12:59:24.2183304
// The current temperature is 76 degrees.
{% endhighlight %}

You can separate the call to WaitAndApologizeAsync from the application of an await operator, as the following code shows. However, remember that a Task doesn't have a Result property, and that no value is produced when an await operator is applied to a Task.

{% highlight csharp %}
Task waitAndApologizeTask = WaitAndApologizeAsync();

string output =
    $"Today is {DateTime.Now:D}\n" +
    $"The current time is {DateTime.Now.TimeOfDay:t}\n" +
    "The current temperature is 76 degrees.\n";

await waitAndApologizeTask;
Console.WriteLine(output);
{% endhighlight %}

#### Task\<TResult\> return type

The Task\<TResult\> return type is used for an async method that contains a return statement in which the operand is TResult.

{% highlight csharp %}
public static async Task ShowTodaysInfoAsync()
{
    string message =
        $"Today is {DateTime.Today:D}\n" +
        "Today's hours of leisure: " +
        $"{await GetLeisureHoursAsync()}";

    Console.WriteLine(message);
}

static async Task<int> GetLeisureHoursAsync()
{
    DayOfWeek today = await Task.FromResult(DateTime.Now.DayOfWeek);

    int leisureHours =
        today is DayOfWeek.Saturday || today is DayOfWeek.Sunday
        ? 16 : 5;

    return leisureHours;
}
// Example output:
//    Today is Wednesday, May 24, 2017
//    Today's hours of leisure: 5
{% endhighlight %}

#### Task.FromResult

Task.FromResult用于创建一个已经完成的Task对象，并包含指定的结果值。这在模拟异步操作或包装同步结果为异步接口时非常有用。

Task.FromResult是轻量级的，不会创建新线程，而是直接返回一个已完成的任务。

{% highlight csharp %}
// 创建一个已完成的任务，包含结果值42
Task<int> completedTask = Task.FromResult(42);

// 使用await获取结果
int result = await completedTask;
Console.WriteLine(result); // 输出: 42
{% endhighlight %}

主要特点：

立即完成：返回的任务状态为 RanToCompletion
无异步操作：不涉及线程池或异步执行
结果可用：任务创建时即包含指定值

典型使用场景：
实现异步接口时包装同步结果
单元测试中模拟异步方法
与现有异步代码集成

性能优化：
对常用值（如 true/false）会缓存任务实例
减少内存分配，提高性能

{% highlight csharp %}
// 同步方法
int ComputeValue() => 100;

// 包装为异步方法
Task<int> ComputeValueAsync() => Task.FromResult(ComputeValue());
{% endhighlight %}

与 Task.Run 的区别：

Task.Run 在线程池中异步执行
Task.FromResult 立即完成 不会创建新的线程


#### Void return type

void 返回类型的使用场景

事件处理程序：异步事件处理程序必须使用 void 返回类型，因为事件处理程序的签名通常要求返回 void。

非事件处理程序：对于不返回值的方法，应使用 Task 作为返回类型，而不是 void，因为 void 返回类型的方法无法被 await。

void 返回类型的问题

无法等待：调用 void 返回类型的异步方法时，调用者无法等待该方法完成，必须独立于该方法的执行继续运行。

异常处理：调用者无法捕获 void 返回类型的异步方法抛出的异常，未处理的异常可能导致应用程序崩溃。

Task 和 Task\<TResult\> 的优势

异常处理：如果返回 Task 或 Task\<TResult\> 的异步方法抛出异常，异常会被存储在返回的任务中，并在任务被 await 时重新抛出。

可等待性：调用者可以等待这些方法的完成，并处理返回的值或异常。

示例说明

异步事件处理程序：在示例代码中，异步事件处理程序需要通知主线程它已完成，以便主线程可以在退出程序之前等待事件处理程序完成。

{% highlight csharp %}
// 异步事件处理程序
async void OnButtonClick(object sender, EventArgs e)
{
    try
    {
        await SomeAsyncOperation();
    }
    catch (Exception ex)
    {
        // 处理异常
    }
    finally
    {
        // 通知主线程完成
    }
}
{% endhighlight %}

{% highlight csharp %}
public class NaiveButton
{
    public event EventHandler? Clicked;

    public void Click()
    {
        Console.WriteLine("Somebody has clicked a button. Let's raise the event...");

        // Invoke 是用于触发事件的方法。
        // this 是事件的发送者（sender），表示触发事件的对象。
        // 这段代码的作用是触发 Clicked 事件，并通知所有订阅了该事件的事件处理程序。
        Clicked?.Invoke(this, EventArgs.Empty);
        Console.WriteLine("All listeners are notified.");
    }
}

public class AsyncVoidExample
{
    static readonly TaskCompletionSource<bool> s_tcs = new TaskCompletionSource<bool>();

    public static async Task MultipleEventHandlersAsync()
    {
        Task<bool> secondHandlerFinished = s_tcs.Task;

        var button = new NaiveButton();

        button.Clicked += OnButtonClicked1;
        button.Clicked += OnButtonClicked2Async;
        button.Clicked += OnButtonClicked3;

        Console.WriteLine("Before button.Click() is called...");
        button.Click();
        Console.WriteLine("After button.Click() is called...");

        await secondHandlerFinished;
    }

    private static void OnButtonClicked1(object? sender, EventArgs e)
    {
        Console.WriteLine("   Handler 1 is starting...");
        Task.Delay(100).Wait();
        Console.WriteLine("   Handler 1 is done.");
    }

    private static async void OnButtonClicked2Async(object? sender, EventArgs e)
    {
        Console.WriteLine("   Handler 2 is starting...");
        Task.Delay(100).Wait();
        Console.WriteLine("   Handler 2 is about to go async...");
        await Task.Delay(500);
        Console.WriteLine("   Handler 2 is done.");
        s_tcs.SetResult(true);
    }

    private static void OnButtonClicked3(object? sender, EventArgs e)
    {
        Console.WriteLine("   Handler 3 is starting...");
        Task.Delay(100).Wait();
        Console.WriteLine("   Handler 3 is done.");
    }
}
// Example output:
//
// Before button.Click() is called...
// Somebody has clicked a button. Let's raise the event...
//    Handler 1 is starting...
//    Handler 1 is done.
//    Handler 2 is starting...
//    Handler 2 is about to go async...
//    Handler 3 is starting...
//    Handler 3 is done.
// All listeners are notified.
// After button.Click() is called...
//    Handler 2 is done.
{% endhighlight %}

#### Generalized async return types and ValueTask\<TResult\>

广义异步返回类型

核心机制：异步方法可以返回任何具有可访问的 GetAwaiter 方法的类型。

要求：

GetAwaiter 方法返回的等待器类型必须实现 System.Runtime.CompilerServices.ICriticalNotifyCompletion 接口。

返回的类型必须具有 System.Runtime.CompilerServices.AsyncMethodBuilderAttribute 属性。

作用：使编译器能够生成返回不同类型（不仅仅是 Task 或 Task\<TResult\>）的异步方法。

性能优化

问题：Task 和 Task\<TResult\> 是引用类型，在性能关键路径（如紧密循环）中频繁分配内存会影响性能。

解决方案：通过广义异步返回类型，可以返回轻量级的值类型（如 ValueTask\<TResult\>），减少内存分配。

ValueTask\<TResult\>

定义：System.Threading.Tasks.ValueTask\<TResult\> 是 .NET 提供的一个轻量级任务返回结构。

优势：

减少堆内存分配。

适合在性能关键场景中替代 Task\<TResult\>。

{% highlight csharp %}
class Program
{
    static readonly Random s_rnd = new Random();

    static async Task Main() =>
        Console.WriteLine($"You rolled {await GetDiceRollAsync()}");

    static async ValueTask<int> GetDiceRollAsync()
    {
        Console.WriteLine("Shaking dice...");

        int roll1 = await RollAsync();
        int roll2 = await RollAsync();

        return roll1 + roll2;
    }

    static async ValueTask<int> RollAsync()
    {
        await Task.Delay(500);

        int diceRoll = s_rnd.Next(1, 7);
        return diceRoll;
    }
}
// Example output:
//    Shaking dice...
//    You rolled 8
{% endhighlight %}


#### Async streams with IAsyncEnumerable\<T\>

异步流（Async Streams）

核心机制：异步方法可以返回 IAsyncEnumerable<T>，表示一个异步流。

作用：提供一种按需生成和消费异步数据的方式，适合处理分块生成的数据流。

使用场景

分块数据：当数据以分块形式生成时（如分页查询、实时数据流）。

异步操作：每次生成数据时可能涉及异步操作（如网络请求、数据库查询）。

示例代码

{% highlight csharp %}
public async IAsyncEnumerable<int> GenerateAsyncStream()
{
    for (int i = 0; i < 10; i++)
    {
        await Task.Delay(100); // 模拟异步操作
        yield return i; // 生成数据
    }
}
{% endhighlight %}

消费异步流

使用 await foreach 消费异步流中的数据：

{% highlight csharp %}
await foreach (var item in GenerateAsyncStream())
{
    Console.WriteLine(item);
}
{% endhighlight %}

总结

异步流：通过 IAsyncEnumerable<T> 实现按需生成和消费异步数据。

适用场景：适合处理分块生成的数据流，如分页查询、实时数据推送等。

优势：减少内存占用，支持异步操作，提升性能。

通过异步流，开发者可以高效处理分块生成的异步数据，特别适合需要流式处理数据的场景。

{% highlight csharp %}
static async IAsyncEnumerable<string> ReadWordsFromStreamAsync()
{
    string data =
        @"This is a line of text.
              Here is the second line of text.
              And there is one more for good measure.
              Wait, that was the penultimate line.";

    using var readStream = new StringReader(data);

    string? line = await readStream.ReadLineAsync();
    while (line != null)
    {
        foreach (string word in line.Split(' ', StringSplitOptions.RemoveEmptyEntries))
        {
            yield return word;
        }

        line = await readStream.ReadLineAsync();
    }
}
{% endhighlight %}