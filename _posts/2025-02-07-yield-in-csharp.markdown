---
layout: post
title:  "Yield keyword in C#"
date:   2025-02-09 00:00:00 -0500
categories: C#
---

The yield statement in an iterator to provide the next value or signal the end of the iteration. The yield statement has the two following forms:

yield return: to provide the next value in the iteration

{% highlight csharp %}
foreach (int i in ProduceEvenNumbers(9))
{
    Console.Write(i);
    Console.Write(" ");
}
// Output: 0 2 4 6 8

IEnumerable<int> ProduceEvenNumbers(int upto)
{
    for (int i = 0; i <= upto; i += 2)
    {
        // yield return 实现了延迟执行
        // 每次循环时生成一个偶数 而不是一次性生成所有数字
        // 当 i 超过upto时 自动结束迭代
        yield return i;
        // 自动封装：每个yield return语句都会向自动生成的集合中添加元素
        // 编译器会生成一个隐藏类来维护循环状态（比如当前i的值)
        // 延迟执行：只有在遍历时才会逐个生成值，而不是一次性生成所有元素
    }
}

// 内存高效	每次只会生成一个值
// 自动实现迭代器模式 编译器会生成状态机来维护迭代进度
// 返回类型必须是IEnumerable 或者 IEnumerator
{% endhighlight %}

yield break: to explicitly signal the end of iteration, as the following example shows

{% highlight csharp %}
Console.WriteLine(string.Join(" ", TakeWhilePositive(new int[] {2, 3, 4, 5, -1, 3, 4})));
// Output: 2 3 4 5

Console.WriteLine(string.Join(" ", TakeWhilePositive(new int[] {9, 8, 7})));
// Output: 9 8 7

IEnumerable<int> TakeWhilePositive(IEnumerable<int> numbers)
{
    foreach (int n in numbers)
    {
        if (n > 0)
        {
            yield return n;
        }
        else
        {
            yield break; // 此处立即终止整个迭代器
            
            // 一旦遇到非正数（0或负数），立即执行 yield break
            // 终止整个迭代器
            // 后续的循环不会继续执行
            // 迭代器状态机被立即销毁
            //与普通break的区别：如果这里使用普通break，只会跳出当前foreach循环，但迭代器会继续执行方法中后续的代码（如果有），而yield break会直接终止整个迭代器。
        }
    }
}
{% endhighlight %}

#### IAsyncEnumerable<T> 是 C# 8.0 引入的异步流处理接口，用于处理异步数据序列

{% highlight csharp %}
// 同步版本（传统集合）
IEnumerable<int> GetNumbers()
{
    for(int i = 0; i < 10; i++)
    {
        Thread.Sleep(100); // 模拟同步阻塞
        yield return i;
    }
}

// 异步版本（异步流）
async IAsyncEnumerable<int> GetNumbersAsync()
{
    for(int i = 0; i < 10; i++)
    {
        await Task.Delay(100); // 模拟异步操作（如网络请求）
        yield return i; // 支持异步的 yield return
    }
}
{% endhighlight %}

主要特性：

异步迭代： 支持await foreach 消费数据

非阻塞处理： 适合网络请求/数据库查询等I/O密集操作

按需生成： 数据按需生成 不占用大量的内存

典型使用场景：

数据库分页查询结果流式处理

实时数据源 

分页调用外部API获取数据

消费事例：

{% highlight csharp %}
await foreach (var number in GetNumbersAsync())
{
    Console.WriteLine($"Received: {number} at {DateTime.Now:HH:mm:ss}");
    // 可以在此处理每个异步到达的数据
}
{% endhighlight %}