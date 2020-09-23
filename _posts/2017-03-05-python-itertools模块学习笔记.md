---
title: python itertools模块学习笔记
date: 2017-03-05 21:50:00 +08:00
modified: 2017-03-05 21:50:00 +08:00
tags: [itertools, python]
---

# python itertools模块学习笔记

## count

原型：

```
count(start[, step])
```

返回：

```
start, start+step, start+2*step, ... (以start为首项, step为公差的等差数列)
```

example:

```
count(10):10, 11, 12, 13, .....
```

## cycle

原型：

```
cycle(p)
```

返回：

```
p0, p1, p2, ...,plast, p0, p1, ... (对序列循环遍历)
```

## repeat

原型：

```
repeat(elem[, n])
```

返回：

```
elem, elem, elem...( 重复n次或者无限下去)
```

## chain

原型：

```
chain(p, q, ...)
```

返回：

```
p0, p1, ..., plast, q0, q1, ..., qlast, ... (一个一个地遍历这些序列的, 不必生成大序列即可这样遍历)
```

## compress

原型:

```
compress(data d, selector s)
```

返回：

```
(d[0] if s[0]), (d[1] if s[1]), ...
```

例子：

```
from itertools import compress

for i in compress("abcdef", [1, 1, 0, 1, 0, 1]):  # a, b, d, f
    print(i)
```

## dropwhile

原型：

```
dropwhile(func f, seq q)
```

返回：

当函数 f 执行返回假时, 开始迭代序列

例子：

```
from itertools import dropwhile

for i in dropwhile(lambda x: x < 5, [1, 2, 6, 4, 3]):  # 6, 4, 3
    print(i)
```

## groupby

原型：

```
groupby(iterable[, keyfunc])
```

返回：按照 keyfunc 函数对序列每个元素执行后的结果分组（每个分组是一个迭代器），返回这些分组的迭代器

例子：

```
from itertools import groupby

a = ['aa', 'ab', 'abc', 'bcd', 'abcde']
for i, k in groupby(a, len): #  按照字符串的长度对a的每个元素进行分组
    for m in k:
        print(m, end=' '), 
    print(i)
```

输出：

```
aa ab 2
abc bcd 3
abcde 5
```

## ifilterfalse

原型：

```
ifilterfalse(func, seq)
```

返回：对函数func执行返回假的元素的迭代器

## islice

原型：

```
islice(seq[, start], stop[, step])
```

返回：返回序列seq的从start开始到stop结束的步长为step的元素的迭代器

```python
from itertools import islice

for i in islice("abcdef", 0, 4, 2): # a, c
    print(i)
```

## imap

原型：

```
imap(func, seq)
```

返回：返回序列每个元素被 func 执行后返回值的序列的迭代器

备注：在 python3 中 map 相当于 itertools.imap  的迭代函数。

```
from itertools import imap

for i in imap(lambda x:x+1, range(10)):#1, 2, ..., 10
    print(i)
```

## starup

原型：

```
starup(func, seq)
```

返回：对序列 seq 的每个元素作为 func 的参数列表执行, 返回执行结果的迭代器

例子：

```
from itertools import starmap

for i in starmap(pow, [(2,2), (3, 2)]): #  4, 9
    print(i)
```

## tee

原型：

```
tee(it[, n = 2])
```

把一个迭代器分为n个迭代器, 返回一个元组.默认是两个

```
from itertools import tee

a = "hello"
c, d = tee(iter(a), 2)
for i, j in zip(c, d):
    print(i, j)
```

输出：

```
h h
e e
l l
l l
o o
```

## takewhile

原型：

```
takewhile(func, seq)
```

从序列的头开始, 直到执行函数func失败.

例子：

```
from itertools import takewhile

for i in takewhile(lambda x: x < 6, range(10)):  #  1, 2, 3, 4, 5
    print(i)
```

## izip

雷同于zip

原型：

```
izip(iter1, iter2, ... iterN):
```

返回：(it1[0], it2 [0], it3[0], ..), (it1[1], it2[1], it3[1], ..)...

只要提供的某个迭代器不再生成值，迭代就会停止

```
izip_longest
```

原型同izip

只不过, 迭代过程会持续到所有输入迭代变量iter1,iter2等都耗尽为止，如果没有使用fillvalue关键字参数指定不同的值，则使用None来填充已经使用的迭代变量的值

## permutations

原型：

```
permutations(p[, r])
```

返回 p 中任意取 r 个元素做排列的元组的迭代器

实例：

```
from itertools import permutations

for i in permutations([1, 2, 3], 3):
    print(i)
```

输出：

```
(1, 2, 3)
(1, 3, 2)
(2, 1, 3)
(2, 3, 1)
(3, 1, 2)
(3, 2, 1)
```

## product

原型：

```
product(iter1, iter2, ... iterN, [repeat=1])
```

创建一个迭代器，生成表示item1，item2等中的项目的笛卡尔积的元组，repeat是一个关键字参数，指定重复生成序列的次数

```python
from itertools import product

for i in product([1, 2, 3], [4, 5], [6, 7]):
    print(i)
```

输出:

```bash
(1, 4, 6)
(1, 4, 7)
(1, 5, 6)
(1, 5, 7)
(2, 4, 6)
(2, 4, 7)
(2, 5, 6)
(2, 5, 7)
(3, 4, 6)
(3, 4, 7)
(3, 5, 6)
(3, 5, 7)
```

## combinations()

原型

```
combinations(iterable, r)
```

创建一个迭代器，返回 iterable 中所有长度为r的子序列，返回的子序列中的项按输入iterable中的顺序排序。

子序列不重复。

例子：

```python
from itertools import combinations

for i in combinations([1, 2, 3], 2):
    print(i)
```

输出：

```
(1, 2)
(1, 3)
(2, 3)
```

## `combinations_with_replacement`

同上，子序列重复：

```python
from itertools import combinations_with_replacement

for i in combinations_with_replacement([1, 2, 3], 2):
    print(i)
```

输出：

```
(1, 1)
(1, 2)
(1, 3)
(2, 2)
(2, 3)
(3, 3)
```



## 备注

**copy from**：<http://outofmemory.cn/code-snippet/2390/python-itertools-module-learn-note>