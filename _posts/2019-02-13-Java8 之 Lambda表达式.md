---
layout:     post
title:      Java8 之 Lambda表达式
subtitle:   关于 Java8 新特性 Lambda表达式 的学习笔记
date:       2019-02-13
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - java8
---

# Java8 之 Lambda表达式

Java8 中的 Lambda表达式 需要“函数式”接口的支持；

## 1.函数式接口

函数式接口 ：只有一个抽象方法的接口；

### 1.1 常用的函数式接口

- 消费型接口：

Consumer<T>
```
    void accept(T t);
```


- 供给型接口：

Supplier<T>
```
    T get();
```


- 函数型接口：

Function<T, R>
```
    R apply(T t);
```

- 断言型接口：

Predicate<T>
```
    boolean test(T t);
```

其他函数式接口可在 [java.util.function](https://docs.oracle.com/javase/8/docs/api/) 包中获取学习；

### 1.2 自定义函数式接口

```
@FunctionalInterface
public interface XXX<T , ...> {

    XXX xxx(T t , ...);
}
```

注意：
- 使用 @FunctionalInterface 标识类，表示这是一个函数式接口，内部只能存在一个抽象方法；

## 2.Lambda表达式

结构 : (...) -> {...}

->  Lambda操作符 （或称为 剪刀操作符 ）

左边  入参

右边  Lambda 体，即 Lambda 表达式要执行的功能

### 2.1 案例

Consumer<String> consumer = (t) -> System.out.println(t);

Supplier<String> supplier = () -> "hello";

Function<String, String> function = (t) -> t + "world";

Predicate<Integer> predicate = (t) -> t >= 60;

## 3.类型推断

Lambda 表达式中无需指定类型，程序依然可以编译，这是因为 javac 根据程序的上下文，在后台推断出了参数的类型。 Lambda 表达式的类型依赖于上下文环境，是由编译器推断出来的。这就是所谓的“类型推断”。

注意：
- Lambda表达式 与 函数式接口 中的方法，入参和出参都是一一对应的；
- 左边只有一个入参时，() 可以不写；
- 右边代码块只有一句时，{} 可以不写；
- 入参的类型是由JVM自动推断得到的；
