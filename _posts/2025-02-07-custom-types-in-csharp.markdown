---
layout: post
title:  "Custom types in C#"
date:   2025-02-11 00:00:00 -0500
categories: C#
---

You use the struct, class, interface, enum, and record constructs to create your own custom types. The .NET class library itself is a collection of custom types that you can use in your own applications. By default, the most frequently used types in the class library are available in any C# program. Others become available only when you explicitly add a project reference to the assembly that defines them. After the compiler has a reference to the assembly, you can declare variables (and constants) of the types declared in that assembly in source code.

One of the first decisions you make when defining a type is deciding which construct to use for your type. The following list helps make that initial decision. There's overlap in the choices. In most scenarios, more than one option is a reasonable choice.

<span style="color: green;">
If the data storage size is small, no more than 64 bytes, choose a struct or record struct.
</span>

<span style="color: green;">
If the type is immutable, or you want nondestructive mutation, choose a struct or record struct.
</span>

<span style="color: green;">
If your type should have value semantics for equality, choose a record class or record struct.
</span>

<span style="color: green;">
If the type is primarily used for storing data, not behavior, choose a record class or record struct.
</span>

<span style="color: green;">
If the type is part of an inheritance hierarchy, choose a record class or a class.
</span>

<span style="color: green;">
If the type uses polymorphism, choose a class.
</span>

<span style="color: green;">
If the primary purpose is behavior, choose a class.
</span>

#### The common type system

继承原则

基类与派生类：C# 支持类型继承，派生类（子类）可以继承基类（父类）的成员（方法、属性等），但有部分限制（如私有成员不可继承）。

继承链：支持多层继承（例如：A → B → C），派生类会继承整个继承链中所有基类的成员。

统一基类：所有类型（包括内置类型如 int）最终都继承自 System.Object（C# 关键字 object）。

{% highlight csharp %}
public class Animal { }          // 基类
public class Dog : Animal { }    // 派生类
public class Puppy : Dog { }     // 多层继承
{% endhighlight %}

公共类型系统（CTS）

统一类型体系：CTS 是 .NET 的核心特性，确保所有类型（包括值类型和引用类型）共享统一的继承层次结构。

类型分类：CTS 将类型分为两类：
值类型：直接存储数据（如 int, struct 定义的类型）。
引用类型：存储数据的内存地址（如 class, record 定义的类型）

值类型与引用类型的运行时行为

值类型：

生命周期由作用域决定，超出作用域后立即释放。

适合小型、频繁使用的数据（如坐标、温度值）。

引用类型：

由垃圾回收器（GC）管理内存。

适合大型对象或需要共享数据的场景。

总结

继承：C# 通过继承实现代码复用，所有类型最终源自 object。

CTS：统一类型系统确保 .NET 语言互操作性。

值类型：轻量级、栈分配，适合高效存储简单数据。

引用类型：堆分配、支持复杂对象，由 GC 管理生命周期。

Classes and structs are two of the basic constructs of the common type system in .NET. Each is essentially a data structure that encapsulates a set of data and behaviors that belong together as a logical unit.

A class is a reference type. When an object of the type is created, the variable to which the object is assigned holds only a reference to that memory. When the object reference is assigned to a new variable, the new variable refers to the original object. Changes made through one variable are reflected in the other variable because they both refer to the same data.

A struct is a value type. When a struct is created, the variable to which the struct is assigned holds the struct's actual data. When the struct is assigned to a new variable, it's copied. The new variable and the original variable therefore contain two separate copies of the same data. Changes made to one copy don't affect the other copy.

Record types can be either reference types (record class) or value types (record struct).

类（Class）

用途：建模复杂行为，支持继承和多态。

数据特性：通常存储可变数据（对象创建后数据可修改）。

示例场景：业务逻辑对象、UI 控件、需要继承的实体。

结构体（Struct）

用途：轻量级小型数据结构。

数据特性：存储不可变或极少修改的数据（默认值类型，直接存储值）。

示例场景：坐标点（Point）、颜色值（Color）、简单数值包装。

记录（Record）

用途：不可变数据模型，编译器自动生成辅助成员。

数据特性：存储不可变数据（默认行为），支持值语义比较。

编译器生成成员：

ToString() 方法

值相等性比较（Equals 和 ==）

GetHashCode()

with 表达式（用于创建副本）

示例场景：DTO（数据传输对象）、配置参数、不可变实体。

#### implicit types, anonymous types, and nullable value types

隐式类型变量（var）

用途：使用 var 关键字声明局部变量，由编译器自动推断类型。

特点：

仅适用于局部变量（不能用于类成员）。

编译时确定类型，非动态类型。

{% highlight csharp %}
var number = 10;          // 推断为 int
var text = "Hello";       // 推断为 string
{% endhighlight %}

匿名类型

用途：临时存储一组相关值，无需显式定义类型。

特点：

适用于方法内部临时数据（不对外暴露）。

属性不可修改（只读）。

{% highlight csharp %}
var person = new { Name = "Alice", Age = 30 };
Console.WriteLine(person.Name); // 输出: Alice
{% endhighlight %}

可空值类型（Nullable Value Types）

用途：允许值类型（如 int）为 null。

语法：在值类型后添加 ?（如 int?）。

底层实现：基于 System.Nullable<T> 结构。

典型场景：

数据库交互（字段可能为 null）。

区分未赋值和零值。

{% highlight csharp %}
int? age = null;
if (age.HasValue)
    Console.WriteLine(age.Value);
else
    Console.WriteLine("Age is unknown.");
{% endhighlight %}

#### 编译时类型 vs 运行时类型

在 C# 中，变量的编译时类型和运行时类型可能不同（尤其在多态场景中），理解两者的区别对代码行为分析至关重要。

编译时类型（Compile-Time Type）

定义：变量声明时的类型（即代码中显式指定的类型）。

作用：

方法调用解析：决定调用哪个重载方法。

成员访问：确定可访问的成员（属性、方法等）。

隐式/显式转换：检查类型转换是否合法。

{% highlight csharp %}
object obj = "Hello"; // 编译时类型是 object
Console.WriteLine(obj.Length); // 编译错误：object 类型没有 Length 属性
{% endhighlight %}

运行时类型（Run-Time Type）

定义：变量实际引用的对象类型（内存中对象的真实类型）。

作用：

虚方法调用：动态分派到实际类型的方法。

类型检查：is、as、switch 表达式等运行时类型判断。

反射：通过 GetType() 获取实际类型。

{% highlight csharp %}
  object obj = "Hello"; // 运行时类型是 string
  if (obj is string s)
      Console.WriteLine(s.Length); // 输出: 5
{% endhighlight %}

示例分析

{% highlight csharp %}
class Animal { public virtual void Speak() => Console.WriteLine("Animal"); }
class Dog : Animal { public override void Speak() => Console.WriteLine("Dog"); }

Animal animal = new Dog(); // 编译时类型 Animal，运行时类型 Dog

// 编译时类型决定可访问的方法
animal.Speak(); // 输出: Dog（运行时类型决定实际调用的方法）

// 编译时类型限制成员访问
// animal.Bark(); // 编译错误：Animal 类型没有 Bark 方法

// 运行时类型检查
if (animal is Dog dog)
    dog.Bark(); // 允许调用 Dog 的 Bark 方法
{% endhighlight %}

总结
编译时类型：影响代码的静态分析（如方法重载、成员访问、类型转换合法性）。

运行时类型：决定动态行为（如虚方法调用、类型检查、反射）。

多态性：通过运行时类型实现动态分派，是面向对象编程的核心机制。