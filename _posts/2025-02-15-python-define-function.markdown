---
layout: post
title:  "Python Define Functions"
date:   2025-02-15 00:00:00 -0500
categories: Python
---

#### Defining Functions

We can create a function that writes the Fibonacci series to an arbitrary boundary:

{% highlight python %}
def fib(n):    # write Fibonacci series less than n
    """Print a Fibonacci series less than n."""
    a, b = 0, 1
    while a < n:
        print(a, end=' ')
        a, b = b, a+b
    print()
{% endhighlight %}

Python 函数定义核心规则

定义语法

使用 def 关键字声明函数，后接函数名和参数列表。

函数体需缩进，从下一行开始编写。

{% highlight python %}
   def function_name(parameters):
       """Docstring (可选)"""
       # 函数体
{% endhighlight %}

#### 文档字符串（Docstring）

函数体首行可写字符串字面量作为说明文档。

用于自动生成文档或代码交互浏览，建议为所有函数添加。

{% highlight python %}
   def add(a, b):
       """返回两个数的和"""
       return a + b
{% endhighlight %}

#### 作用域与变量查找

函数执行时创建局部符号表存储局部变量。

#### 变量引用优先级：

局部 → 外层函数 → 全局 → 内置作用域

#### 修改外部变量：

全局变量需用 global 声明。

外层函数变量需用 nonlocal 声明。

#### 参数传递机制

参数按对象引用传递（传递的是引用，非对象值拷贝）。

每次函数调用（包括递归）会创建新的局部符号表。

#### 函数对象与命名

函数名绑定到当前符号表的函数对象。

允许通过其他别名访问同一函数：

{% highlight python %}
  calc = add  # 别名
   print(calc(2, 3))  # 输出 5
{% endhighlight %}

#### global 和 nonlocal 声明

##### global 声明

作用：在函数内部修改全局变量。

{% highlight python %}
x = 10  # 全局变量

def modify_global():
    global x  # 声明使用全局变量 x
    x = 20    # 修改全局变量

modify_global()
print(x)  # 输出 20
{% endhighlight %}

##### nonlocal 声明

作用：在嵌套函数中修改外层（非全局）作用域的变量。

{% highlight python %}
def outer():
    y = 30  # 外层函数的变量

    def inner():
        nonlocal y  # 声明使用外层变量 y
        y = 40      # 修改外层变量

    inner()
    print(y)  # 输出 40

outer()
{% endhighlight %}