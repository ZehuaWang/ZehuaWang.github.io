---
layout: post
title:  "Asynchronous programming in C#"
date:   2025-02-07 00:00:00 -0500
categories: C#
---

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