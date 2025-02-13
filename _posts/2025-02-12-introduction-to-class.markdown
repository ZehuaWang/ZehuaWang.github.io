---
layout: post
title:  "Introduction to Class"
date:   2025-02-10 00:00:00 -0500
categories: C#
---

#### Constructors and initialization


类实例初始化方法

在 C# 中，创建类实例时可以通过以下方式初始化字段和属性的值：

使用默认值

规则：所有 .NET 类型都有默认值。

数值类型（如 int、double）默认为 0。

引用类型（如 string、class）默认为 null。

bool 类型默认为 false。

适用场景：当默认值符合业务逻辑时直接使用。

{% highlight csharp %}
public class Person
{
    public int Age;       // 默认 0
    public string Name;   // 默认 null
}
{% endhighlight %}

字段初始化器

规则：在声明字段时直接赋值。

特点：在构造函数执行前完成初始化。

{% highlight csharp %}
public class Person
{
    public int Age = 18;      // 直接初始化为 18
    public string Name = "Unknown";
}
{% endhighlight %}

构造函数参数

规则：通过构造函数参数初始化字段或属性。

特点：支持动态赋值，灵活性高。

{% highlight csharp %}
public class Person
{
    public int Age;
    public string Name;

    public Person(int age, string name)
    {
        Age = age;
        Name = name;
    }
}
{% endhighlight %}

对象初始化器

规则：在创建对象时通过 {} 语法直接赋值。

特点：无需显式定义构造函数，代码更简洁。

{% highlight csharp %}
var person = new Person
{
    Age = 20,
    Name = "Alice"
};
{% endhighlight %}

#### Class inheritance


C# 的类完全支持继承这一面向对象编程的核心特性。继承允许派生类（子类）复用基类（父类）的成员（字段、属性、方法等），并扩展或修改其行为。

继承的基本语法

{% highlight csharp %}
public class Manager : Employee  // Employee 是基类，Manager 是派生类
{
    // 继承 Employee 的所有成员（构造函数除外）
    // 可添加新成员或重写基类虚方法
}
{% endhighlight %}

继承规则

单继承：一个类只能直接继承一个基类。

间接多继承：通过继承链实现（例如：A → B → C，C 间接继承 A 和 B）。

成员继承：

派生类继承基类的所有成员（字段、属性、方法、事件等）。

例外：构造函数不继承，需在派生类中重新定义或调用基类构造函数。

接口实现

一个类可以实现多个接口，弥补单继承的局限性。

接口定义行为规范，类需实现接口的所有成员。

{% highlight csharp %}
public class Manager : Employee, IWork, IReport  // 继承基类 + 实现两个接口
{
    // 实现 IWork 和 IReport 的成员
}
{% endhighlight %}

抽象类与密封类

抽象类 包含抽象方法（只有签名，无实现）- 不能实例化，只能通过派生类使用

密封类 禁止其他类继承 - 用于防止派生类修改关键逻辑

分部类

用途：将类的定义拆分到多个源文件中。

场景：大型类、自动生成代码与手动编写代码分离。

{% highlight csharp %}
// File1.cs
public partial class Manager
{
    public void Work() { ... }
}

// File2.cs
public partial class Manager
{
    public void Report() { ... }
}
{% endhighlight %}

关键总结

继承：通过 : BaseClass 实现代码复用，支持单继承。

接口：通过 : IInterface 实现多行为规范。

抽象类：定义通用模板，强制派生类实现抽象方法。

密封类：禁止继承，保护核心逻辑。

分部类：拆分代码，提升可维护性。

密封类定义

密封类使用 sealed 关键字声明，禁止其他类继承它。

{% highlight csharp %}
public sealed class EncryptionService
{
    public string Encrypt(string data)
    {
        // 加密逻辑（示例）
        return Convert.ToBase64String(Encoding.UTF8.GetBytes(data));
    }

    public string Decrypt(string encryptedData)
    {
        // 解密逻辑（示例）
        return Encoding.UTF8.GetString(Convert.FromBase64String(encryptedData));
    }
}
{% endhighlight %}

尝试继承密封类（会编译失败）

{% highlight csharp %}
// 错误：无法从密封类 EncryptionService 派生
public class AdvancedEncryptionService : EncryptionService
{
    // 编译错误 CS0509
}
{% endhighlight %}

密封类的典型场景

安全敏感类：防止子类篡改核心逻辑（如加密、身份验证）。

工具类：已完善的功能，无需扩展（如 System.String）。

性能优化：避免虚方法调用的开销（JIT 编译器可内联方法）。

抽象类定义

抽象类使用 abstract 关键字声明，可以包含抽象方法（无实现）和具体方法（有实现）。

{% highlight csharp %}
public abstract class Shape
{
    // 抽象方法（无实现，必须由派生类重写）
    public abstract double Area();

    // 具体方法（有默认实现，可被派生类直接使用或重写）
    public void Display()
    {
        Console.WriteLine($"Shape area: {Area()}");
    }
}
{% endhighlight %}

继承抽象类并实现抽象方法

圆形类（实现 Area 方法）：

{% highlight csharp %}
public class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }

    public override double Area()
    {
        return Width * Height;
    }
}
{% endhighlight %}

使用示例

{% highlight csharp %}
public class Program
{
    public static void Main()
    {
        // 抽象类不能实例化
        // Shape shape = new Shape(); // 编译错误

        // 使用派生类
        Circle circle = new Circle { Radius = 5 };
        circle.Display(); // 输出: Shape area: 78.5398...

        Rectangle rect = new Rectangle { Width = 4, Height = 6 };
        rect.Display();  // 输出: Shape area: 24
    }
}
{% endhighlight %}

关键特性

抽象方法：强制派生类实现特定逻辑（如 Area）。

具体方法：提供通用实现（如 Display）。

不可实例化：只能通过派生类使用。