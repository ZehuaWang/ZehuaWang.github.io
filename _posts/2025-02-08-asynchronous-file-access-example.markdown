---
layout: post
title:  "Asynchronous file access (C#)"
date:   2025-02-11 00:00:00 -0500
categories: C#
---

<span style="color: green;">You can use the async feature to access files. By using the async feature, you can call into asynchronous methods without using callbacks or splitting your code across multiple methods or lambda expressions.</span>

You might consider the following reasons for adding asynchrony to file access calls:

Asynchrony makes UI applications more responsive because the UI thread that launches the operation can perform other work. If the UI thread must execute code that takes a long time (for example, more than 50 milliseconds), the UI may freeze until the I/O is complete and the UI thread can again process keyboard and mouse input and other events.
Asynchrony improves the scalability of ASP.NET and other server-based applications by reducing the need for threads. If the application uses a dedicated thread per response and a thousand requests are being handled simultaneously, a thousand threads are needed. Asynchronous operations often don't need to use a thread during the wait. They use the existing I/O completion thread briefly at the end.
The latency of a file access operation might be very low under current conditions, but the latency may greatly increase in the future. For example, a file may be moved to a server that's across the world.
The added overhead of using the Async feature is small.
Asynchronous tasks can easily be run in parallel.

#### Use appropriate classes

简单示例

文档提到可以使用 File.WriteAllTextAsync 和 File.ReadAllTextAsync 进行简单的异步文件读写操作。这些方法是高层 API，适合处理简单的文件 I/O 任务。

精细控制

对于需要更精细控制的文件 I/O 操作，建议使用 FileStream 类。FileStream 提供了更多选项，可以在操作系统级别实现异步 I/O，从而避免阻塞线程池线程。

启用异步 I/O

为了启用异步 I/O，可以在 FileStream 的构造函数中指定以下参数之一：

useAsync=true

options=FileOptions.Asynchronous

这些选项可以确保 I/O 操作在操作系统级别异步执行，减少对线程池线程的依赖。

StreamReader 和 StreamWriter 的限制

直接使用文件路径：如果直接通过文件路径创建 StreamReader 或 StreamWriter，则无法使用异步 I/O 选项。

通过 FileStream 使用：如果通过 FileStream 创建 StreamReader 或 StreamWriter，则可以启用异步 I/O。

总结
简单操作：使用 File.WriteAllTextAsync 和 File.ReadAllTextAsync。

精细控制：使用 FileStream 并启用 useAsync=true 或 options=FileOptions.Asynchronous。

StreamReader/StreamWriter：通过 FileStream 启用异步 I/O。

UI 应用：异步调用有助于保持 UI 响应性。

{% highlight csharp %}
using System;
using System.IO;
using System.Threading.Tasks;

public class Program
{
    public static async Task Main()
    {
        string filePath = "test.txt";

        // 异步写入文件
        using (FileStream fs = new FileStream(filePath, FileMode.Create, FileAccess.Write, FileShare.None, bufferSize: 4096, useAsync: true))
        {
            byte[] data = System.Text.Encoding.UTF8.GetBytes("Hello, FileStream!");
            await fs.WriteAsync(data, 0, data.Length);
        }

        // 异步读取文件
        using (FileStream fs = new FileStream(filePath, FileMode.Open, FileAccess.Read, FileShare.None, bufferSize: 4096, useAsync: true))
        {
                byte[] buffer = new byte[1024];
                int bytesRead = await fs.ReadAsync(buffer, 0, buffer.Length);
                string content = System.Text.Encoding.UTF8.GetString(buffer, 0, bytesRead);
                Console.WriteLine(content);
        }
    }
}
{% endhighlight %}

#### Parallel asynchronous I/O

The following examples demonstrate parallel processing by writing 10 text files.

{% highlight csharp %}
public static async Task SimpleParallelWriteAsync()
{
    string folder = Directory.CreateDirectory("tempfolder").Name;
    IList<Task> writeTaskList = new List<Task>();

    for (int index = 11; index <= 20; ++ index)
    {
        string fileName = $"file-{index:00}.txt";
        string filePath = $"{folder}/{fileName}";
        string text = $"In file {index}";

        writeTaskList.Add(File.WriteAllTextAsync(filePath, text));
    }

    await Task.WhenAll(writeTaskList);
}
{% endhighlight %}

