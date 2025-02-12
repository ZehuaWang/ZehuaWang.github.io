---
layout: post
title:  "Introduction to delegates and events in C#"
date:   2025-02-10 00:00:00 -0500
categories: C#
---

Delegates provide a late binding mechanism in .NET. Late Binding means that you create an algorithm where the caller also supplies at least one method that implements part of the algorithm.

在 C# 中，委托（Delegate） 是一种类型安全的函数指针，用于封装和传递方法。以下是简要说明：

定义

委托是一种引用类型，用于表示具有特定签名（参数和返回类型）的方法。

可以将方法作为参数传递，或存储在变量中。

特点

类型安全：委托在编译时检查方法的签名。

多播支持：一个委托可以绑定多个方法，按顺序调用。

灵活性：支持回调、事件处理等场景。

声明与使用

{% highlight csharp %}
// 声明委托
public delegate void MyDelegate(string message);
{% endhighlight %}

{% highlight csharp %}
public class Program
{
    public static void Main()
    {
        MyDelegate del = new MyDelegate(ShowMessage);
        del("Hello, Delegate!");
    }

    public static void ShowMessage(string message)
    {
        Console.WriteLine(message);
    }
}
{% endhighlight %}


#### <span color="green">System.Delegate and the delegate keyword</span>

#### Define the delegate type

The code that the compiler generates when you use the delegate keyword will map to method calls that invoke members of the Delegate and MulticastDelegate classes.

You define a delegate type using syntax that is similar to defining a method signature. You just add the delegate keyword to the definition.

Let's continue to use the List.Sort() method as our example. The first step is to create a type for the comparison delegate:

{% highlight csharp %}
public delegate int Comparison<in T>(T left, T right);
{% endhighlight %}

#### Declare instances of delegates

After defining the delegate, you can create an instance of that type. Like all variables in C#, you cannot declare delegate instances directly in a namespace, or in the global namespace.

{% highlight csharp %}
// inside a class definition:

// Declare an instance of that type:
public Comparison<T> comparator;
{% endhighlight %}

<span style="color:green">The type of the variable is Comparison<T>, the delegate type defined earlier. The name of the variable is comparator.</span>

That code snippet above declared a member variable inside a class. You can also declare delegate variables that are local variables, or arguments to methods.

#### Invoke delegate

You invoke the methods that are in the invocation list of a delegate by calling that delegate. Inside the Sort() method, the code will call the comparison method to determine which order to place objects:

{% highlight csharp %}
int result = comparator(left, right);
{% endhighlight %}

In the line above, the code invokes the method attached to the delegate. You treat the variable as a method name, and invoke it using normal method call syntax.

#### Assign, add, and remove invocation targets

Developers that want to use the List.Sort() method need to define a method whose signature matches the delegate type definition, and assign it to the delegate used by the sort method. This assignment adds the method to the invocation list of that delegate object.

Suppose you wanted to sort a list of strings by their length. Your comparison function might be the following:

{% highlight csharp %}
private static int CompareLength(string left, string right) =>
    left.Length.CompareTo(right.Length);
{% endhighlight %}

You create that relationship by passing that method to the List.Sort() method:

{% highlight csharp %}
phrases.Sort(CompareLength);
{% endhighlight %}

You could also have been explicit by declaring a variable of type Comparison<string> and doing an assignment:

{% highlight csharp %}
Comparison<string> comparer = CompareLength;
phrases.Sort(comparer);
{% endhighlight %}

#### Delegate and MulticastDelegate classes

{% highlight csharp %}
class Program
{
    public delegate void MyDelegate(string message);

    public static void ShowMessage(string message)
    {
        Console.WriteLine("ShowMessage: " + message);
    }

    public static void LogMessage(string message)
    {
        Console.WriteLine("LogMessage: " + message);
    }

    public static void DisplayMessage(string message)
    {
        Console.WriteLine("DisplayMessage: " + message);
    }

    public static void Main()
    {
        MyDelegate myDelegate = ShowMessage;
        myDelegate += LogMessage;
        myDelegate += DisplayMessage;

        myDelegate("Hello, World!");
    }
}
{% endhighlight %}

<span style="color:red">Note:</span>

<span style="color:green">in</span> keyword

in 关键字用于定义泛型委托 Comparison<T> 的逆变（Contravariance）特性。以下是详细说明：

逆变：in 关键字表示泛型类型参数 T 是逆变的，即允许使用比指定类型更通用的类型。

参数位置：in 只能用于输入参数（如方法参数），不能用于输出参数或返回值。

代码示例

{% highlight csharp %}
public delegate int Comparison<in T>(T left, T right);
{% endhighlight %}

in T 表示 T 是逆变的，只能用于输入参数（left 和 right）。

这意味着可以将 Comparison<Base> 赋值给 Comparison<Derived> 类型的变量。

逆变的使用场景

灵活性：允许使用更通用的类型，提高代码的灵活性。

委托兼容性：使委托类型更兼容，支持更广泛的类型转换。

示例说明

假设有以下类层次结构：

{% highlight csharp %}
class Animal { }
class Dog : Animal { }
{% endhighlight %}

可以这样使用：

{% highlight csharp %}
Comparison<Animal> animalComparison = (a1, a2) => 0;
Comparison<Dog> dogComparison = animalComparison; // 由于逆变，这是合法的
{% endhighlight %}

编译器行为

当使用 delegate 关键字定义委托时，编译器会自动生成代码，这些代码会映射到 System.Delegate 和 System.MulticastDelegate 类的成员方法调用。

生成的代码结构

假设你定义了一个委托：

{% highlight csharp %}
public delegate void MyDelegate(string message);
{% endhighlight %}

编译器会生成一个继承自 MulticastDelegate 的类：

{% highlight csharp %}
public class MyDelegate : System.MulticastDelegate
{
    public MyDelegate(object target, IntPtr methodPtr);
    public virtual void Invoke(string message);
    public virtual IAsyncResult BeginInvoke(string message, AsyncCallback callback, object state);
    public virtual void EndInvoke(IAsyncResult result);
}
{% endhighlight %}

关键类的作用

Delegate 类：提供基础功能，如目标对象和方法指针的存储。

MulticastDelegate 类：继承自 Delegate，支持多播委托（即一个委托实例可以绑定多个方法）。

方法调用

Invoke 方法：同步调用所有绑定的方法。

BeginInvoke/EndInvoke 方法：提供异步调用支持（注：.NET Core 及以上版本已弃用异步委托调用）。

示例说明

当使用委托调用方法时：

{% highlight csharp %}
MyDelegate del = ShowMessage;
del("Hello");
{% endhighlight %}

编译器会生成类似这样的代码：

{% highlight csharp %}
del.Invoke("Hello");
{% endhighlight %}

多播委托的实现

当委托绑定多个方法时：

{% highlight csharp %}
MyDelegate del1 = ShowMessage;
MyDelegate del2 = LogMessage;
MyDelegate multiDel = del1 + del2;
{% endhighlight %}

总结

delegate 关键字是语法糖，底层依赖于 Delegate 和 MulticastDelegate 类。

编译器生成的代码实现了委托的同步/异步调用和多播功能。

这种设计使得委托在 C# 中既灵活又高效，支持事件处理、回调等场景。

核心机制：

多播支持：通过继承 MulticastDelegate，委托实例可以维护方法调用列表

类型安全：编译器生成的 Invoke 方法强制参数和返回类型匹配

运行时绑定：通过存储目标对象和方法指针实现动态调用

这种设计模式使得委托能够：

支持 += 和 -= 操作符添加/移除方法

实现事件处理机制

作为回调函数的基础设施

