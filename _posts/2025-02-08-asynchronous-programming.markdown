---
layout: post
title:  "Asynchronous programming in C#"
date:   2025-02-07 00:00:00 -0500
categories: C#
---

[Async programming](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)

The Task asynchronous programming model (TAP) provides an abstraction over asynchronous code. You write code as a sequence of statements, just like always. You can read that code as though each statement completes before the next begins. The compiler performs many transformations because some of those statements may start work and return a **Task that represents the ongoing work**.

Enable code that reads like a sequence of statements, but executes in a much more complicated order based on external resource allocation and when tasks are complete.

Cooking breakfast is a good example of asynchronous work that isn't parallel. **One person (or thread) can handle all these tasks**. Continuing the breakfast analogy, one person can make breakfast **asynchronously by starting the next task before the first task completes**.

For a parallel algorithm, you'd need multiple cooks (or threads).

{% highlight csharp %}
using System;
using System.Threading.Tasks;

namespace AsyncBreakfast
{
    // These classes are intentionally empty for the purpose of this example. They are simply marker classes for the purpose of demonstration, contain no properties, and serve no other purpose.
    internal class Bacon { }
    internal class Coffee { }
    internal class Egg { }
    internal class Juice { }
    internal class Toast { }

    class Program
    {
        static void Main(string[] args)
        {
            Coffee cup = PourCoffee();
            Console.WriteLine("coffee is ready");

            Egg eggs = FryEggs(2);
            Console.WriteLine("eggs are ready");

            Bacon bacon = FryBacon(3);
            Console.WriteLine("bacon is ready");

            Toast toast = ToastBread(2);
            ApplyButter(toast);
            ApplyJam(toast);
            Console.WriteLine("toast is ready");

            Juice oj = PourOJ();
            Console.WriteLine("oj is ready");
            Console.WriteLine("Breakfast is ready!");
        }

        private static Juice PourOJ()
        {
            Console.WriteLine("Pouring orange juice");
            return new Juice();
        }

        private static void ApplyJam(Toast toast) =>
            Console.WriteLine("Putting jam on the toast");

        private static void ApplyButter(Toast toast) =>
            Console.WriteLine("Putting butter on the toast");

        private static Toast ToastBread(int slices)
        {
            for (int slice = 0; slice < slices; slice++)
            {
                Console.WriteLine("Putting a slice of bread in the toaster");
            }
            Console.WriteLine("Start toasting...");
            Task.Delay(3000).Wait();
            Console.WriteLine("Remove toast from toaster");

            return new Toast();
        }

        private static Bacon FryBacon(int slices)
        {
            Console.WriteLine($"putting {slices} slices of bacon in the pan");
            Console.WriteLine("cooking first side of bacon...");
            Task.Delay(3000).Wait();
            for (int slice = 0; slice < slices; slice++)
            {
                Console.WriteLine("flipping a slice of bacon");
            }
            Console.WriteLine("cooking the second side of bacon...");
            Task.Delay(3000).Wait();
            Console.WriteLine("Put bacon on plate");

            return new Bacon();
        }

        private static Egg FryEggs(int howMany)
        {
            Console.WriteLine("Warming the egg pan...");
            Task.Delay(3000).Wait();
            Console.WriteLine($"cracking {howMany} eggs");
            Console.WriteLine("cooking the eggs ...");
            Task.Delay(3000).Wait();
            Console.WriteLine("Put eggs on plate");

            return new Egg();
        }

        private static Coffee PourCoffee()
        {
            Console.WriteLine("Pouring coffee");
            return new Coffee();
        }
    }
}
{% endhighlight %}

Computers don't interpret those instructions the same way people do. **The computer will block on each statement until the work is complete before moving on to the next statement**.

#### <span style="color: red;">Don't block, await instead</span>

As written, this code blocks the thread executing it from doing any other work. It won't be interrupted while any of the tasks are in progress. It would be as though you stared at the toaster after putting the bread in. **You'd ignore anyone talking to you until the toast popped** (The thread is blocked and waiting the current task or method to complete).

Let's start by updating this code so that the thread doesn't block while tasks are running. The **await keyword provides a non-blocking way to start a task, then continue execution when that task completes**.

{% highlight csharp %}
static async Task Main(string[] args)
{
    Coffee cup = PourCoffee();
    Console.WriteLine("coffee is ready");

    Egg eggs = await FryEggsAsync(2);
    Console.WriteLine("eggs are ready");

    Bacon bacon = await FryBaconAsync(3);
    Console.WriteLine("bacon is ready");

    Toast toast = await ToastBreadAsync(2);
    ApplyButter(toast);
    ApplyJam(toast);
    Console.WriteLine("toast is ready");

    Juice oj = PourOJ();
    Console.WriteLine("oj is ready");
    Console.WriteLine("Breakfast is ready!");
}
{% endhighlight %}

<span style="color: green;">The total elapsed time is roughly the same as the initial synchronous version. The code has yet to take advantage of some of the key features of asynchronous programming.</span>

note: 通过使用await 线程不会被阻塞 可以去执行其他的任务 但是不会继续执行当前的方法知道返回了结果 比如厨师可以去帮助别人 但是不会继续煎蛋直到培根做好了

这种方法不能充分利用异步编程的优势 但是在gui编程中 通过使用await关键字 可以避免ui阻塞

<span style="color: green;">You don't want each of the component tasks to be executed sequentially. It's better to start each of the component tasks before awaiting the previous task's completion.</span>

#### <span style="color: red;">Start tasks concurrently</span>

In many scenarios, you want to start several independent tasks immediately. Then, as each task finishes, you can continue other work that's ready. 在许多情况下 我们想要同时启动所有的task 当每个task完成时 我们可以继续执行接下来的业务代码

The System.Threading.Tasks.Task and related types are classes you can use to reason about tasks that are in progress. 我们可以使用Tasks类来代表正在进行的任务

{% highlight csharp %}
Coffee cup = PourCoffee();
Console.WriteLine("Coffee is ready");

Task<Egg> eggsTask = FryEggsAsync(2);
Task<Bacon> baconTask = FryBaconAsync(3);
Task<Toast> toastTask = ToastBreadAsync(2);

Toast toast = await toastTask;
ApplyButter(toast);
ApplyJam(toast);
Console.WriteLine("Toast is ready");
Juice oj = PourOJ();
Console.WriteLine("Oj is ready");

Egg eggs = await eggsTask;
Console.WriteLine("Eggs are ready");
Bacon bacon = await baconTask;
Console.WriteLine("Bacon is ready");

Console.WriteLine("Breakfast is ready!");
{% endhighlight %}

The preceding code works better. **You start all the asynchronous tasks at once. You await each task only when you need the results.** The preceding code may be similar to code in a web application that makes requests to different microservices, then combines the results into a single page. You'll make all the requests immediately, then await all those tasks and compose the web page. (比如一个web应用 需要请求不同的微服务 然后组合成一个页面 我们可以立即开始所有的请求 然后等待所有的请求完成 然后组合成一个页面)

note: <span style="color: green;">这种方法可以充分利用异步编程的优势 但是需要等待所有task完成 才能继续执行接下来的业务代码</span>

#### <span style="color: red;">Composition with tasks</span>

Making the toast is the composition of an asynchronous operation (toasting the bread), and synchronous operations (adding the butter and the jam). Updating this code illustrates an important concept. 烤面包包括一个异步的过程和一个同步的过程

note: <span style="color: green;">The composition of an asynchronous operation followed by synchronous work is an asynchronous operation. Stated another way, if any portion of an operation is asynchronous, the entire operation is asynchronous.</span>

{% highlight csharp %}
static async Task<Toast> MakeToastWithButterAndJamAsync(int number)
{
    var toast = await ToastBreadAsync(number);
    ApplyButter(toast);
    ApplyJam(toast);

    return toast;
}
{% endhighlight %}

The preceding method has the async modifier in its signature. That signals to the compiler that this method contains an await statement; it contains asynchronous operations. This method represents the task that toasts the bread, then adds butter and jam. This method returns a Task<TResult> that represents the composition of those three operations. 

前面的代码使用async来修饰这个方法 表示这个方法包含await语句 包含异步操作 这个方法表示烤面包 然后添加黄油和果酱 这个方法返回一个Task<TResult> 表示这三个操作的组合

{% highlight csharp %}
static async Task Main(string[] args)
{
    Coffee cup = PourCoffee();
    Console.WriteLine("coffee is ready");

    var eggsTask = FryEggsAsync(2);
    var baconTask = FryBaconAsync(3);
    var toastTask = MakeToastWithButterAndJamAsync(2);

    var eggs = await eggsTask;
    Console.WriteLine("eggs are ready");

    var bacon = await baconTask;
    Console.WriteLine("bacon is ready");

    var toast = await toastTask;
    Console.WriteLine("toast is ready");

    Juice oj = PourOJ();
    Console.WriteLine("oj is ready");
    Console.WriteLine("Breakfast is ready!");
}
{% endhighlight %}

**The previous change illustrated an important technique for working with asynchronous code. You compose tasks by separating the operations into a new method that returns a task. You can choose when to await that task. You can start other tasks concurrently.**

note: 前面的代码演示了异步编程的一个重要技术 通过将操作分离成一个返回任务的新方法 可以选择何时等待该任务 可以并发启动其他任务

#### <span style="color: red;">Asynchronous exceptions</span>

Asynchronous methods throw exceptions, just like their synchronous counterparts. Asynchronous support for exceptions and error handling strives for the same goals as asynchronous support in general: You should write code that reads like a series of synchronous statements. Tasks throw exceptions when they can't complete successfully. The client code can catch those exceptions when a started task is awaited.

When a task that runs asynchronously throws an exception, that Task is faulted. The Task object holds the exception thrown in the Task.Exception property. Faulted tasks throw an exception when they're awaited.

The Task.Exception property is a System.AggregateException because more than one exception may be thrown during asynchronous work. Any exception thrown is added to the AggregateException.InnerExceptions collection.

The most common scenario for a faulted task is that the Exception property contains exactly one exception. When code awaits a faulted task, the first exception in the AggregateException.InnerExceptions collection is rethrown.

#### <span style="color: red;">Await tasks efficiently</span>

The series of await statements at the end of the preceding code can be improved by using methods of the Task class. One of those APIs is WhenAll, which returns a Task that completes when all the tasks in its argument list have completed.

{% highlight csharp %}
await Task.WhenAll(eggsTask, baconTask, toastTask);
Console.WriteLine("Eggs are ready");
Console.WriteLine("Bacon is ready");
Console.WriteLine("Toast is ready");
Console.WriteLine("Breakfast is ready!");
{% endhighlight %}

Another option is to use WhenAny, which returns a Task<Task> that completes when any of its arguments complete. You can await the returned task, knowing that it has already finished.

{% highlight csharp %}
var breakfastTasks = new List<Task> { eggsTask, baconTask, toastTask };
while (breakfastTasks.Count > 0)
{
    Task finishedTask = await Task.WhenAny(breakfastTasks);
    if (finishedTask == eggsTask)
    {
        Console.WriteLine("Eggs are ready");
    }
    else if (finishedTask == baconTask)
    {
        Console.WriteLine("Bacon is ready");
    }
    else if (finishedTask == toastTask)
    {
        Console.WriteLine("Toast is ready");
    }
    await finishedTask;
    breakfastTasks.Remove(finishedTask);
}
{% endhighlight %}

Near the end, you see the line await finishedTask;. **The line await Task.WhenAny doesn't await the finished task. It awaits the Task returned by Task.WhenAny. The result of Task.WhenAny is the task that has completed (or faulted). You should await that task again, even though you know it's finished running**. That's how you retrieve its result, or ensure that the exception causing it to fault gets thrown.

#### Task WhenAll API

使用场景如下

并行执行多个独立的任务

当需要发起多个互不依赖的异步操作时 用Task.WhenAll并行处理 可以大幅度减少总耗时

场景

同步调用多个API
批量读取 写入文件
并发数据库查询

{% highlight csharp %}
var task1 = DownloadFileAsync("url1");
var task2 = DownloadFileAsync("url2");
var task3 = DownloadFileAsync("url3");

// 并行等待所有下载完成
var results = await Task.WhenAll(task1, task2, task3);
Console.WriteLine($"下载了 {results.Length} 个文件");
{% endhighlight %}

组合多个任务结果

当需要合并多个任务的结果时 使用Task.WhenAll

{% highlight csharp %}

async Task<int> GetStockAsync(string symbol) { /*...*/ }

var tasks = new List<Task<int>> 
{
    GetStockAsync("AAPL"),
    GetStockAsync("MSFT"),
    GetStockAsync("GOOGL")
};

int[] prices = await Task.WhenAll(tasks);
int total = prices.Sum(); // 直接使用所有结果
{% endhighlight %}

处理异常

当需要捕获多个任务中可能出现的所有异常时

{% highlight csharp %}
try
{
    await Task.WhenAll(task1, task2, task3);
}
catch (AggregateException ae)
{
    foreach (var ex in ae.InnerExceptions)
    {
        Console.WriteLine($"错误: {ex.Message}");
    }
}
{% endhighlight %}

优化资源的利用率

避免用await逐个等待任务 防止线程阻塞

{% highlight csharp %}
// ❌ 低效写法（串行）
var result1 = await task1;
var result2 = await task2;

// ✅ 高效写法（并行）
await Task.WhenAll(task1, task2);
{% endhighlight %}