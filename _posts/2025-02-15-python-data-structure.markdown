---
layout: post
title:  "Data Structure in Python"
date:   2025-02-15 00:00:00 -0500
categories: Python
---

#### More on Lists

#### 列表方法分类

修改列表

append(x) 在列表末尾添加元素 x [1,2].append(3) → [1,2,3]

extend(iterable) 将可迭代对象的所有元素追加到列表末尾 [1].extend([2,3]) → [1,2,3]

insert(i, x) 在索引 i 前插入元素 x [1,3].insert(1,2) → [1,2,3]

remove(x) 删除第一个值为 x 的元素（找不到则抛 ValueError） [1,2,3].remove(2) → [1,3]

pop([i]) 删除并返回索引 i 处的元素（默认删除最后一个元素，空列表抛 IndexError）[1,2,3].pop(1) → 2（列表变为 [1,3]）

clear() 删除列表所有元素 [1,2,3].clear() → []

reverse() 反转列表元素 [1,2,3].reverse() → [3,2,1]

sort(key, reverse) 原地排序（参数同 sorted()）[3,1,2].sort() → [1,2,3]

查找元素

index(x[, start[, end]]) 返回第一个值为 x 的元素的索引（可指定搜索范围 [start, end)，找不到抛 ValueError）['a','b','a'].index('a',1) → 2

count(x) 返回 x 在列表中出现的次数 [1,2,2,3].count(2) → 2

复制

copy() 返回列表的浅拷贝（等价于 a[:]）b = a.copy()

关键对比

append(x) vs extend(iterable) append 添加单个元素，extend 合并可迭代对象元素

pop() vs remove() pop 按索引删除并返回元素，remove 按值删除首个匹配项

sort() vs sorted() sort 原地排序，sorted 返回新列表

reverse() vs reversed() reverse 原地反转，reversed 返回迭代器

注意事项:

原地操作：sort()、reverse()、append() 等方法直接修改原列表，无返回值。

浅拷贝陷阱：copy() 和 a[:] 仅复制列表本身，嵌套的可变对象（如子列表）仍共享引用。

异常处理：使用 remove()、index()、pop() 时建议捕获 ValueError/IndexError。

#### Use List as Stack

LIFO 原则：后进先出（Last-In-First-Out），最后添加的元素最先被移除。

入栈（Push） append(x) O(1) 将元素 x 添加到栈顶

出栈（Pop）pop() O(1) 移除并返回栈顶元素（无参数）

{% highlight python %}
# 初始化栈
stack = [3, 4, 5]

# 入栈操作
stack.append(6)  # 栈变为 [3, 4, 5, 6]
stack.append(7)  # 栈变为 [3, 4, 5, 6, 7]

# 出栈操作
print(stack.pop())  # 输出 7 → 栈变为 [3, 4, 5, 6]
print(stack.pop())  # 输出 6 → 栈变为 [3, 4, 5]
print(stack.pop())  # 输出 5 → 栈变为 [3, 4]
{% endhighlight %}

注意事项

性能：列表的 append 和 pop 操作在尾部进行，时间复杂度为 O(1)，适合实现栈。

替代方案：对于高频栈操作，可用 collections.deque 优化性能（线程安全且高效）。

错误处理：空栈调用 pop() 会抛出 IndexError，需提前检查栈是否为空。

#### collections.deque

collections.deque（双端队列）的用法详解

基本特性

高效操作：两端（头部/尾部）的插入和删除时间复杂度为 O(1)，中间操作为 O(n)。

线程安全：适用于多线程环境。

固定大小：可通过 maxlen 参数限制容量（自动丢弃旧元素）。

核心方法

作为栈（LIFO）

入栈（Push）append(x) d.append(3) → 尾部添加元素

出栈（Pop）pop() d.pop() → 移除并返回尾部元素

作为队列（FIFO）

入队（Enqueue）append(x) d.append(3) → 尾部添加元素

出队（Dequeue）popleft() d.popleft() → 移除并返回头部元素

其他操作

appendleft(x) 头部添加元素 d.appendleft(0)

extend(iterable) 尾部批量添加元素 d.extend([4,5])

extendleft(iterable) 头部批量添加元素（逆序插入）d.extendleft([1,2]) → 插入后顺序为 2,1

rotate(n) 循环右移 n 步（负值左移）d.rotate(1) → [5,1,2,3,4]

使用示例

初始化与基本操作

{% highlight python %}
from collections import deque

# 创建 deque（可指定初始元素和最大长度）
d = deque([1, 2, 3], maxlen=5)  # 最大长度 5，超长时自动丢弃旧元素

# 添加元素
d.append(4)       # 尾部添加 → deque([1, 2, 3, 4])
d.appendleft(0)   # 头部添加 → deque([0, 1, 2, 3, 4])

# 移除元素
last = d.pop()    # 移除尾部元素 4 → last=4
first = d.popleft()  # 移除头部元素 0 → first=0

print(d)  # 输出 deque([1, 2, 3])
{% endhighlight %}

固定长度示例

{% highlight python %}
d = deque(maxlen=3)
d.append(1)  # deque([1])
d.append(2)  # deque([1, 2])
d.append(3)  # deque([1, 2, 3])
d.append(4)  # deque([2, 3, 4])（自动丢弃头部元素 1）
{% endhighlight %}

旋转操作

{% highlight python %}
d = deque([1, 2, 3, 4, 5])
d.rotate(2)   # 右移 2 步 → deque([4, 5, 1, 2, 3])
d.rotate(-1)  # 左移 1 步 → deque([5, 1, 2, 3, 4])
{% endhighlight %}

总结

优先使用 deque：当需要高频操作头部/尾部元素时（如队列、栈）。

避免中间操作：insert(index, x) 或 remove(x) 效率较低（O(n)）。

适用场景：消息队列、滑动窗口算法、撤销操作历史等。