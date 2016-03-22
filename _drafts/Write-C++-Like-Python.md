---
title: 像 Python 一样写 C++
layout: post
category: ModernCpp
---

## Pythonic
> 人生苦短，我用 Python

Python 内置了一些非常有用的函数，可以让你写出 Pythonic 的代码，简洁高效。  

举个栗子：
Q: 返回 1~9 的每个元素的立方组成的列表。
不熟悉 `map` 的人可能会写出如下的代码：

```Python
cubes = []

def cube(x):
    return x**3

for x in range(1, 10):
    cubes.append(cube(x))

```

用 `map` 只需要一行代码。

```Python
cubes = map(lambda x: x**3, range(1, 10))
```

除了 map 和 lambda 表达式，Python 还内置了诸如 filter、map、reduce 等函数，组合使用，使用很少的代码，就能实现复杂的功能。  

C++ 能不能也写得如此优雅呢？

## Modern C++
> “C++11 feels like a new language.” – Bjarne Stroustrup




