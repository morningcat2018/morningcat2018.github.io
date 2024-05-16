---
layout:     post
title:      python的那些小知识
subtitle:   语法知识笔记
date:       2024-04-30
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - python
---


## 类变量 类方法 和 静态方法

```py
"""
类变量 类方法 和 静态方法
https://www.perry.qa/morse
"""


class Student:
    student_count = 0  # 类变量

    def __init__(self, name):
        self.name = name
        Student.student_count += 1

    @classmethod  # 类方法
    def add_count(cls, count):
        cls.student_count += count

    @classmethod
    def get_count(cls):
        return cls.student_count

    @classmethod
    def build(cls, str):
        return cls(str.split(" ")[0])

    @staticmethod  # 静态方法
    def build_stu(name):
        return Student(name)


if __name__ == '__main__':
    p = Student("morningcat")
    print("student count:{}".format(Student.student_count))
    Student.add_count(100)
    print("student count:{}".format(Student.get_count()))
    p = Student.build("Hello world")
    print(p.name)
```

## 魔法方法

```py
"""
魔法方法:特殊方法
通常写在类定义里,用来定义类如何与内置函数和运算符交互
更多魔法方法:https://diveintopython3.net/special-method-names.html

"""


class Order:

    def __init__(self, count):
        self.count = count

    def __add__(self, other):
        return Order(self.count + other.count)

    def __str__(self):
        return "Order({})".format(self.count)

    def __len__(self):
        return self.count

    def __call__(self, add_count):
        self.count += add_count


if __name__ == '__main__':
    # 同等效果
    x1 = 1 + 2
    x2 = (x := 1).__add__(2)

    # 自定义类
    a = Order(12)
    b = Order(25)
    order_c = a + b
    print(order_c)
    order_c(100)
    print(order_c)
```

## 可迭代对象

```python
"""
可迭代对象, 以及迭代器
"""


class LineIterator:

    def __init__(self, filepath):
        self.file = open(filepath, 'r')

    def __iter__(self):
        return self

    def __next__(self):
        line = self.file.readline()
        if line:
            return line
        else:
            self.file.close()
            raise StopIteration("file read over")


if __name__ == '__main__':
    # 常见可迭代对象
    s1 = "hello"
    print(hasattr(s1, "__iter__"))
    for x in s1:
        print(x)

    l1 = [1, 2, 3]
    print(hasattr(l1, "__iter__"))
    for x in l1:
        print(x)

    num1 = 133
    print(hasattr(num1, "__iter__"))

    # 模拟迭代原理
    it = iter(l1)
    while True:
        try:
            print(next(it))
        except StopIteration:
            break

    fileIter = LineIterator("/home/1.txt")
    for line in fileIter:
        print(line)
```



## 推导式

```py
"""
推导式: 可高效获取可迭代对象(list dict set)
"""

if __name__ == '__main__':
    # 列表推导式
    list1 = range(11)
    list2 = [x ** 2 for x in list1 if x % 2 == 0]
    print(list2)

    letters = ['a', 'b', 'c']
    nums = [1, 2, 3]
    list3 = [(i, j) for i in letters for j in nums]
    print(list3)

    # 字典推导式
    dict1 = {i: j for i, j in zip(letters, nums)}
    print(dict1)

    # 集合推导式
    set1 = {x for x in letters}
    print(set1 )
```

## 生成器

```py
"""
生成器
"""


def square_numbers(nums):
    """
    包含yield关键字,即是一个生成器函数
    """
    for i in nums:
        yield i ** 2


if __name__ == '__main__':
    gen = square_numbers(range(5))
    print(gen)
    print(next(gen))
    for x in gen:
        print(x)

    # 生成器
    gen2 = (x ** 2 for x in range(5))
    print(gen2) 
```

## 装饰器

```py
"""
装饰器
"""
import functools
import time


def timer(threshold):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            start = time.time()
            result = func(*args, **kwargs)
            end = time.time()
            if end - start > threshold:
                raise Exception(f"timeout {end - start}")
            return result

        return wrapper

    return decorator


@timer(0.1)  # 0.1 second
def test1():
    time.sleep(0.4)


if __name__ == '__main__':
    test1()
```

## lambda函数

```
"""
匿名函数 : lambda函数
"""


def user_login(user):
    level_credit_dict = {
        1: lambda x: x + 2,
        2: lambda x: x + 5,
        3: lambda x: x + 10,
    }
    user.credit = level_credit_dict[user.level](user.credit)


if __name__ == '__main__':
    add = lambda a, b: a + b
    print(add(1, 3))

    list1 = [1, 2, 3, 4, 5]
    list2 = list(map(lambda x: x ** 2, list1))
    print(list2)
```
