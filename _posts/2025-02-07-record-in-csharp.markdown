---
layout: post
title:  "Record keyword in C#"
date:   2025-02-07 00:00:00 -0500
categories: C#
---

### Record in C#

Record modifire to define a reference type that provides built-in functionality for encapsulating data.

record class and record struct


{% highlight csharp %}
public record Person(string FirstName, string LastName);
{% endhighlight %}

{% highlight csharp %}
public record Person
{
    public required string FirstName { get; init; }
    public required string LastName { get; init; }
};
{% endhighlight %}

init 是 C# 9.0 引入的一个新特性，它是一个只能在对象初始化时设置值的访问器

主要特点：
1. 初始化后不可变
    属性只能在对象构造时或对象初始化器中设置
    设置后就不能再修改（实现了不可变性）

2. 使用示例：

{% highlight csharp %}
// 创建对象时可以设置值
var person = new Person 
{ 
    FirstName = "张",  // 可以在初始化时设置
    LastName = "三"    // 可以在初始化时设置
};

// 之后尝试修改会导致编译错误
person.FirstName = "李";  // 编译错误！init 属性不能在初始化后修改
{% endhighlight %}

3. 与 readonly 的区别:
    readonly 只能在构造函数中设置
    init 可以在对象初始化器中设置

4. 常见用途：
    创建不可变对象
    确保对象创建后的数据完整性
    线程安全性（因为不可变）

这确保了 Person 记录一旦创建后，其 FirstName 和 LastName 就不能被修改，保持了数据的不可变性。

The following two examples demonstrate record struct value types:

{% highlight csharp %}
public record struct Point(int X, int Y);
{% endhighlight %}

{% highlight csharp %}
public record struct Point
{
    public required int X { get; init; }
    public required int Y { get; init; }
};
{% endhighlight %}

You can also create records with mutable properties and fields:


{% highlight csharp %}
public record Person
{
    public required string FirstName { get; set; }
    public required string LastName { get; set; }
};
{% endhighlight %}

Record structs can be mutable as well

{% highlight csharp %}
public record struct Point
{
    public int X { get; set; }
    public int Y { get; set; }
};
{% endhighlight %}

While records can be mutable, they're primarily intended for **supporting immutable data models**.

The record type offers the following features:

- Concise syntax for creating a reference type with immutable properties
- Built-in behavior useful for a data-centric reference type:
    - Value equality
    - Concise syntax for nondestructive mutation
    - Built-in formatting for display
    - Support for inheritance hierarchies

Examples of using **record class**:

Basic record declaration

{% highlight csharp %}
public record Person
{
    public string FirstName { get; init; } // Immutable property
    public string LastName { get; init; }
}

// Usage
var person1 = new Person { FirstName = "Alice", LastName = "Smith" };
Console.WriteLine(person1); // Output: Person { FirstName = Alice, LastName = Smith }
{% endhighlight %}

Value based equality

{% highlight csharp %}
var person1 = new Person { FirstName = "Alice", LastName = "Smith" };
var person2 = new Person { FirstName = "Alice", LastName = "Smith" };
Console.WriteLine(person1 == person2); // Output: True, Compares by value not by reference
{% endhighlight %}

Positional record declaration

{% highlight csharp %}
public record Book(string Title, string Author, int Pages);

// Usage (constructor-like syntax)
var book = new Book("The Hobbit", "J.R.R. Tolkien", 310);
Console.WriteLine(book); // Output: Book { Title = The Hobbit, Author = J.R.R. Tolkien, Pages = 310 }
{% endhighlight %}

Non-destructive mutation (with keyword)

{% highlight csharp %}
var originalBook = new Book("The Hobbit", "J.R.R. Tolkien", 310);
var updatedBook = originalBook with { Pages = 320 };

Console.WriteLine(updatedBook); // Pages changed to 320
{% endhighlight %}

在 C# 中，非破坏性变异（Non-destructive mutation）是指在不改变原始对象的情况下创建一个新对象，并对其进行修改。这种特性在 C# 的 record 类型中通过 with 表达式实现.

优势：

数据完整性：原始对象不被修改，确保数据的完整性。

线程安全：由于对象不可变，减少了多线程环境下的数据竞争问题。

简化代码：通过 with 表达式，代码更简洁易读。

这种特性特别适合需要频繁更新但又希望保持原始数据不变的场景。

Descturction support

{% highlight csharp %}
var (title, author, pages) = updatedBook; // Deconstruct into variables
Console.WriteLine($"Title: {title}, Author: {author}");
{% endhighlight %}

Inheritance with records

{% highlight csharp %}
public record Vehicle(string Manufacturer, int Wheels);
public record Car(string Manufacturer, int Wheels, string FuelType) : Vehicle(Manufacturer, Wheels);

// Usage
Vehicle car = new Car("Tesla", 4, "Electric");
Console.WriteLine(car); // Output: Car { Manufacturer = Tesla, Wheels = 4, FuelType = Electric }
{% endhighlight %}

Records with methods

{% highlight csharp %}
public record Rectangle(int Width, int Height)
{
    public int Area => Width * Height; // Computed property
    public void Display() => Console.WriteLine($"Rectangle {Width}x{Height}");
}

// Usage
var rect = new Rectangle(10, 20);
Console.WriteLine(rect.Area); // Output: 200
rect.Display(); // Output: Rectangle 10x20
{% endhighlight %}

Records for DTOs

{% highlight csharp %}
public record ApiResponse(bool Success, string Data, DateTime Timestamp);

// Simulate an API response
var response = new ApiResponse(true, "{ 'id': 42 }", DateTime.UtcNow);
Console.WriteLine(response);
{% endhighlight %}


Examples of using **record struct**:


When to use record keyword:

使用场景：  
1. 不可变对象：
当你需要创建不可变对象时，record 是一个很好的选择。它们的属性可以使用 init 访问器，使对象在初始化后不可变。
2. 数据传输对象 (DTO)：
record 非常适合用作数据传输对象，因为它们可以轻松地表示数据结构，并且支持值相等性。
3. 值相等性：
如果你需要对象的相等性基于值而不是引用，record 提供了内置的值相等性支持。
4. 简洁的语法：
record 提供了简洁的语法来定义数据模型，尤其是使用位置记录（positional records）时。
5. 非破坏性变异：
当你需要在不改变原始对象的情况下创建一个修改后的新对象时，record 的 with 表达式非常有用。
模式匹配：
record 类型与模式匹配结合得很好，适合在需要解构对象的场景中使用。

Link: [Record in C#](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record)