---
title: functools.partial 的使用技巧
date: 2018-10-20 18:45:00 +08:00
modified: 2018-10-20 18:45:00 +08:00
tags: [partial, python]
---

# functools.partial 的使用技巧

functools.partial 可以用来改变一个方法默认参数

1. 改变原有默认值参数的默认值
2. 给原来没有默认值的参数增加默认值

#### 使用方法举例：

```python
from functools import partial

def spam(a, b, c, d):
    print(a, b, c, d)
s1 = partial(spam, 1)
s1(2,3,4)  # 1 2 3 4
s1(4,5,6)  # 1 4 5 6
s2 = partial(spam, d=42) 
s2(4, 5, 6)  # 4 5 6 42
s3 = partial(spam, 1, 2, d=42)
s3(3)  # 1 2 3 42
s3(4)  # 1 2 4 42
s3(5)  # 1 2 5 42
```

一个计算相对距离的例子：

```python
points = [(1,2), (3,4),(5,6),(7,8)]

import math
def distance(p1, p2):
    x1, y1 = p1
    x2, y2 = p2
    return math.hypot(x2 - x1, y2 - y1)

pt=(4, 3)
points.sort(key=partial(distance, pt))
print(points)  # [(3, 4), (1, 2), (5, 6), (7, 8)]
```



####  更新partial对象的 \_\_name\_\_  \_\_doc\_\_ 属性

```bash
def foo(a, b=0):
    '''
   	int add
    '''
    print(a + b)

foo2 = functools.partial(foo, a=10)
foo2()  # 10
```



**foo2 ** 是一个partial 对象，它只有3个只读属性，如下：

```
print(foo2.func)
print(foo2.args)
print(foo2.keywords)
print(dir(foo2))
```
默认情况下 partial 对象是没有 __name__ __doc__ 属性，使用 update_wrapper 从原始方法中添加属性到 partial 对象中

```bash
print(foo2.__doc__)
partial(func, *args, **keywords) - new function with partial application
    of the given arguments and keywords.
```

修改为foo的文档信息了

```python
functools.update_wrapper(foo2, foo)  # functools.partial(<function foo at 0x111310b00>, a=10)
print(foo2.__doc__)  # int add
```

