---
layout: post
title: python import 机制
date: 2017-05-14 12:00
modified: 2017-05-14 12:00
description: python import 机制
tag:
  - import
  - python
  - software
---


# python import 机制

此文讨论 pyhton3 中 import 机制及使用注意事项，参考目录结构如下：

```
package/
├── __init__.py
├── moduleA.py
├── subpackage1
│   ├── __init__.py
│   ├── __pycache__
│   │   └── moduleX.cpython-37.pyc
│   ├── moduleX.py
│   └── moduleY.py
└── subpackage2
    ├── __init__.py
    └── moduleZ.py
```

> 注意：python2 优先搜索当前路径，也就是 relative-then-absolute 顺序；python3 移除了该设定，严格按照 sys.path 进行搜索。



## 模块(module) 和 包(package)

一般情况下，是一个以.py为后缀的文件。其他可作为module的文件类型还有”.pyo”、”.pyc”、”.pyd”、”.so”、”.dll”，

为避免模块名冲突，python 引入了按目录组织模块的方法，称之为包（package）。包是含有Python模块的文件夹。


## import 是如何确定当前搜索路径的？

导入所使用的模块名不包含路径信息，这需要系统提供搜索方式和匹配规则。其中搜索路径，由解释器在启动时，按优先级整理到 sys.path 列表中。

搜索路径列表：

1. 程序根目录。

2. 环境变量（PYTHONPATH）设定的路径列表。

3. 标准库目录。

4. 第三方扩展库等附加路径（site-specific）。

> 附加路径由 site 模块添加，在解释器启动时自动执行（可用参数 -S 禁用）。
>
> 除系统 site-packages 外，还包括用户相关目录。

```bash
$ python -m site    # 查看完整搜索路径列表
sys.path = [
    '/Users/g/Desktop/python_demos',    # 应用程序根目录
    '/Users/g/.pyenv/versions/3.7.3/lib/python37.zip',    # 标准库
    '/Users/g/.pyenv/versions/3.7.3/lib/python3.7',
    '/Users/g/.pyenv/versions/3.7.3/lib/python3.7/lib-dynload',
    '/Users/g/.pyenv/versions/3.7.3/lib/python3.7/site-packages',    # 系统扩展库目录
    '/Users/g/.pyenv/versions/3.7.3/lib/python3.7/site-packages/pyinotify-0.9.6-py3.7-macosx-10.14-x86_64.egg',
]
USER_BASE: '/Users/g/.local' (exists)
USER_SITE: '/Users/g/.local/lib/python3.7/site-packages' (doesn't exist)
ENABLE_USER_SITE: True

```



## 绝对导入和相对导入

> 相对导入只能用于 from 子句，其可导入模块或成员。
>
> 单个  '.' 表示当前包，'..' 表示上级包， '...' 表示更上一级的包，以此类推。

**入口文件和模块**

简单来说，直接运行 python 文件（`python myfile.py`）与把 python 文件 import 到某个文件再执行有很大的不同。仅仅知道文件位于那个目录，并不能确定它也位于package相同位置，这取决于我们是如何加载 python 文件的（通过直接运行文件 `python myfile.py` 还是 import）。

有两种加载 python 文件的方式：作为入口文件或作为模块。如果直接运行文件（例如，在命令行中键入`python myfile.py`），文件将作为入口文件加载。如果你执行`python -m myfile`，或者在其他文件中使用 import 语句将其加载，文件将作为模块加载。入口文件同一时间只运行一个，它是作为启动 python 程序的文件。



**命名空间**

当 python 文件被加载时，它会有一个 `__name__` 属性。如果它被当作入口文件加载，它的 `__name__` 属性为 `__main__`；当作为模块加载时，`__name__`属性为 `模块名.子模块名.文件名` ，通过点`.`分割。

当你导入 `moduleX` 后，它的 `__name__` 属性值为 `package.subpackage1.moduleX`；当你导入 `moduleA` 后，它的 `__name__` 属性值为 `package.moduleA`。  然而，当你直接在命令行中运行 `moduleX` 后，它的 `__name__` 属性值将为变为 `__main__`；当你直接在命令行运行 `moduleA` 后，它的 `__name__` 属性值将变为 `__main__`；当作为入口文件直接运行时，文件的 `__name__` 属性值将变为 `__main__` 。



**直接访问包内模块**

这里有一点小小的困惑：当你直接从当前文件夹或者从包里导入模块的时候， 模块的 `__name__` 属性是不同的。

例如：如果你在当前文件夹 `package/subpackage1` 运行 python 解释器并导入`moduleX`模块，它的`__name__`值为`moduleX`，而不是`package.subpackage1.moduleX`。这是因为当 python 解释器启动时，它会把当前目录作为它的第一优先级搜索路径。当解释器发现当前目录就有它需要的模块时，它不会知道此目录也是包的一部分，而包的信息也不会出现在模块的`__name__`值中。

我们可以得出一个结论：如果导入模块的`__name__`值中没有分割点，那么它也就不是包的一部分。不管文件在磁盘上的实际位置，它的`__name__`值决定了解释器如何导入它。

我们看下 PEP 328 中的一段话，翻译过来是：

> Relative imports use a module's name attribute to determine that module's position in the package hierarchy. If the module's name does not contain any package information (e.g. it is set to 'main') then relative imports are resolved as if the module were a top level module, regardless of where the module is actually located on the file system.
>
> 相对导入通过模块的 `__name__` 值来确定模块在包层次结构中的位置。如果该模块的`__name__`值不含任何包信息（例如，它的值被设为 `__main__`），那么相对引用会认为这个模块就是顶级模块，而忽略模块在文件系统上的实际位置。



**相对导入**

相对导入使用模块的`__name__`值来确定它在包中的位置。当你使用相对导入语句`from .. import foo`，两个原点表示在包层次结构中上一级的包（单个原点表示当前包，两个表示上级包，三个表示更上一级的包，以此类推）。例如：如果你的当前目录是`package.subpackage1.moduleX`，那么`..moduleA`表示`package.moduleA`。为了让`from .. import `能工作，模块`__name__`值中的点不能少于`import`表达式中的点。



**相对导入只用于包中**

然而，如果模块的`__name__`值为`__main__`，它就不会是一个包。它的`__name__`中没有点，因此它不能使用`from .. import `表达式。如果你这样做，它会报错`relative-import in non-package`。



**入口文件不能使用相对导入**

你可能会在命令行中直接运行 `moduleX` 模块。当你这样做时，它的`__name__`值会变成`__main__`，这意味着文件中的相对导入会失败，因为它的`__name__`中不含包的信息。

同样请记住，当我们交互式的运行python解释器的时候，它的`__name__`值总是为`__main__`，因此你不能在交互式python解释器中使用相对导入。相对导入只用于模块之间。



## 参考文档

[Python中import的用法](https://zhuanlan.zhihu.com/p/63143493)
[Python 的 Import 陷阱](https://medium.com/pyladies-taiwan/python-的-import-陷阱-3538e74f57e3)
[PEP 328 -- Imports: Multi-Line and Absolute/Relative](https://www.python.org/dev/peps/pep-0328/)  
https://docs.python.org/3.7/tutorial/modules.html
https://docs.python.org/3/reference/import.html
[Absolute vs Relative Imports in Python](https://realpython.com/absolute-vs-relative-python-imports/)  
https://stackoverflow.com/questions/14132789/relative-imports-for-the-billionth-time/14132912#14132912
https://vimiix.com/post/2017/12/29/import-error-relative-no-parent/
https://stackoverflow.com/questions/30669474/beyond-top-level-package-error-in-relative-import
https://stackoverflow.com/questions/35166821/valueerror-attempted-relative-import-beyond-top-level-package
https://stackoverflow.com/questions/6323860/sibling-package-imports
https://www.jianshu.com/p/5cc20b88bcf4
