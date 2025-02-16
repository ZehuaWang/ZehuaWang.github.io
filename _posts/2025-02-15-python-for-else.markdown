---
layout: post
title:  "For else in Python"
date:   2025-02-15 00:00:00 -0500
categories: Python
---

#### Python for-else 结构详解

一、基本语法

{% highlight python %}
for item in iterable:
    # 循环体
    if condition:
        break
else:
    # 循环结束后执行的代码
{% endhighlight %}

二、工作原理

正常结束：当循环完整遍历所有元素后（未触发break），执行else块

中断退出：如果循环中执行了break语句，else块将被跳过

三、典型应用场景

质数判断（原始示例）

{% highlight python %}
for n in range(2, 10):
    for x in range(2, n):
        if n % x == 0:
            print(f"{n} = {x}×{n//x}")
            break
    else:
        print(f"{n} 是质数")
{% endhighlight %}

元素搜索

{% highlight python %}
names = ["Alice", "Bob", "Charlie"]
target = "David"

for name in names:
    if name == target:
        print("找到目标人员")
        break
else:
    print("未找到指定人员")
# 输出：未找到指定人员
{% endhighlight %}

数据验证

{% highlight python %}
numbers = [2, 4, 6, 8, 10]

for num in numbers:
    if num % 2 != 0:
        print("发现非偶数")
        break
else:
    print("所有数字均为偶数")
# 输出：所有数字均为偶数
{% endhighlight %}

资源清理

{% highlight python %}
for line in open("data.txt"):
    if line.startswith("ERROR"):
        print("发现错误日志")
        break
else:
    print("未发现错误日志")
# 无论是否发现错误，最后都会执行
print("文件处理完成")
{% endhighlight %}

注意事项

非必须结构：else块不是必须的，只在需要后续处理时使用

与if的区别：else属于for循环，不是与循环内的if配对

执行流程示意图

开始循环
↓
遍历所有元素 → 遇到break → 跳过else
↓          ↖         ↗
正常结束 → 执行else

进阶用法

结合异常处理：

{% highlight python %}
for url in url_list:
    try:
        response = requests.get(url)
        if response.status_code == 200:
            print("成功获取响应")
            break
    except RequestException:
        continue
else:
    raise ConnectionError("所有URL访问失败")
{% endhighlight %}