---
layout: post
title:  "Async programming scenario in C#"
date:   2025-02-09 00:00:00 -0500
categories: C#
---

[Async programming model](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/task-asynchronous-programming-model)

You can avoid performance bottlenecks and enhance the overall responsiveness of your application by using asynchronous programming.


#### Async improves responsiveness

Asynchrony is essential for activities that are potentially blocking, such as web access. Access to a web resource sometimes is slow or delayed. If such an activity is blocked in a synchronous process, the entire application must wait. In an asynchronous process, the application can continue with other work that doesn't depend on the web resource until the potentially blocking task finishes.

如果这个操作是阻塞的 例如 访问一个慢的资源 那么整个应用必须等待 如果这个操作是异步的 那么应用可以继续其他工作 直到阻塞的操作完成

Note

<span style="color: green;">
The async-based approach adds the equivalent of an automatic transmission to the list of options that you can choose from when designing asynchronous operations. That is, you get all the benefits of traditional asynchronous programming but with much less effort from the developer.
</span>

#### Async methods are easy to write

The async and await keywords in C# are the heart of async programming.

{% highlight csharp %}
public async Task<int> GetUrlContentLengthAsync()
{
    using var client = new HttpClient();

    Task<string> getStringTask =
        client.GetStringAsync("https://learn.microsoft.com/dotnet");

    DoIndependentWork();

    string contents = await getStringTask;

    return contents.Length;
}

void DoIndependentWork()
{
    Console.WriteLine("Working...");
}
{% endhighlight %}

How to make an async method:

The method signature must include the `async` modifier.

The name of an async method, by convention, ends with an `Async` suffix.

The return type is one of the following types:

- Task<TResult> if your method has a return statement in which the operand has type TResult.

- Task if your mehtod has no return statement or has a return statement with no operand.

- void if you are writing an async event handler.

- Any other type that has a GetAwaiter method.

<span style="color: green;">
The method usually includes at least one await expression, which marks a point where the method can't continue until the awaited asynchronous operation is complete. In the meantime, the method is suspended, and control returns to the method's caller. 
</span>

#### What happens in an async method

The most important thing to understand in asynchronous programming is how the control flow moves from method to method.

![Control flow](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/media/task-asynchronous-programming-model/navigation-trace-async-program.png#lightbox)


#### API async methods

You can recognize them by the "Async" suffix that's appended to the member name, and by their return type of Task or Task<TResult>. For example, the System.IO.Stream class contains methods such as CopyToAsync, ReadAsync, and WriteAsync alongside the synchronous methods CopyTo, Read, and Write.

#### Threads

Async methods are intended to be non-blocking operations. An await expression ion an async method does not block the current thread while the awaited task is running. 

The `async` and `await` keywords do not cause additional threads to be created. Async methods do not require multiple threads because an async method does not run on its own thread. <span style="color: green;">You can use `Task.Run` to move CPU-bound work to a background thread, but a background thread does not help with a process that's just waiting for results to become available.</span>

#### Task.Run API

在C#中 `Task.Run` 是一个将同步代码异化的关键API 可以将一段代码（同步或者异步）包装成一个Task 并且调度到线程池中执行 从而避免阻塞当前的线程 如UI线程或者主线程

基本作用

- 将代码提交到线程池 将同步或者异步操作包装成`Task` 由线程池分配后台线程执行
- 避免阻塞当前线程 防止耗时操作卡住主线程 提升程序响应性能
- 简化异步编程 为同步代码提供异步执行的快捷方式

{% highlight csharp %}
// 示例：将同步方法异步化
var result = await Task.Run(() => {
    // 这是一个同步的耗时操作（如复杂计算）
    return CalculateSomething();
});
{% endhighlight %}

典型使用场景

场景 1 ： CPU密集型操作

将需要大量计算的代码 例如数据处理 图像渲染 移到后台线程 避免卡住主线程

{% highlight csharp %}
// UI 应用中保持界面响应
button.Click += async (sender, e) => {
    // 在后台线程执行计算
    var result = await Task.Run(() => HeavyCpuWork());
    UpdateUI(result);
};
{% endhighlight %}

场景 2 ： 混合异步和同步代码

当调用一个不支持异步的同步API时，使用Task.Run包装它：

{% highlight csharp %}
// 假设 LegacyCode.SlowMethod() 是同步的
var data = await Task.Run(() => LegacyCode.SlowMethod());
{% endhighlight %}

场景 3 ： 并行执行多个任务

结合 `Task.WhenAll` 并行执行多个任务 提高效率

{% highlight csharp %}
var task1 = Task.Run(() => ProcessDataA());
var task2 = Task.Run(() => ProcessDataB());
await Task.WhenAll(task1, task2);
{% endhighlight %}

注意事项

不要滥用 `Task.Run`

I/O操作不要需要`Task.Run` 直接使用异步API 如 `HttpClient.GetStringAsync`

{% highlight csharp %}
// 错误用法：I/O 操作直接用异步 API，不要用 Task.Run！
  var data = await Task.Run(() => httpClient.GetAsync("url"));

  // 正确用法
  var data = await httpClient.GetAsync("url");
{% endhighlight %}

ASP.NET 慎用 否则会浪费线程池资源 降低性能

异常处理

`Task.Run`返回的Task会封装异常 需要通过try/catch或者await捕获

{% highlight csharp %}
try {
      await Task.Run(() => ThrowException());
  } catch (Exception ex) {
      // 处理异常
  }
{% endhighlight %}

取消操作

结合 `CancellationToken` 实现取消操作

{% highlight csharp %}
var cts = new CancellationTokenSource();
  var task = Task.Run(() => {
      while (true) {
          cts.Token.ThrowIfCancellationRequested();
          // 执行工作
      }
  }, cts.Token);

  // 取消任务
  cts.Cancel();
{% endhighlight %}

Task.Run VS async/await

Task.Run

同步代码异步化 CPU密集型

显式占用线程池线程

可能浪费线程池资源

async/await

原生异步操作 I/O密集型

I/O操作不占用线程 通过回调实现

更高效 I/O操作无线程阻塞

#### async and await

The marked async method can use await to designate suspension points. The await operator tells the compiler that the async method can't continue past that point until the awaited asynchronous process is complete. In the meantime, control returns to the caller of the async method.

The marked async method can itself be awaited by methods that call it.

如果async方法中不包含await 那么它和同步方法没什么区别 不会带来性能提升