---
layout: post
title:  "Anonymous types"
date:   2025-02-13 00:00:00 -0500
categories: C#
---

Anonymous types provide a convenient way to encapsulate a set of read-only properties into a single object without having to explicitly define a type first.

Anonymous types are typically used in the select clause of a query expression to return a subset of the properties from each object in the source sequence.

匿名类型核心特性

成员限制：

仅包含公共只读属性，不可包含方法、事件等其他成员。

属性初始化值不能为 null、匿名函数或指针类型。

命名规则：

若未显式指定属性名，则自动使用初始化表达式的属性名。

支持显式命名（如 new { Name = prod.Color }）。

典型场景：

在 LINQ 查询中选择部分属性，减少数据传输量。

临时存储查询结果，无需定义完整类。

示例说明

定义数据源类

{% highlight csharp %}
class Product
{
    public string? Color { get; set; }
    public decimal Price { get; set; }
    public string? Name { get; set; }
    // 其他属性（如 Category、Size）被忽略
}
{% endhighlight %}

使用匿名类型选择部分属性

{% highlight csharp %}
var productQuery = 
    from prod in products 
    select new { prod.Color, prod.Price }; // 自动生成 Color 和 Price 属性

foreach (var item in productQuery)
{
    Console.WriteLine($"Color={item.Color}, Price={item.Price}");
}
{% endhighlight %}

关键优势

简化代码：无需预先定义类即可封装数据。

提升性能：仅传输必要属性，减少内存占用。

强类型支持：编译器自动生成类型，确保类型安全。

注意事项

匿名类型实例不可修改属性值（所有属性为只读）。

匿名类型作用域限于方法内部，无法作为参数或返回值传递。