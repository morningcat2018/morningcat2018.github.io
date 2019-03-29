---
layout:     post
title:      Java8 之 Optional
subtitle:   Optional容器类 优雅规避空指针
date:       2019-03-29
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - java8
---

## Optional

public final class Optional<T>
extends Object

Optional<T>类是一个容器类，代表一个值存在或不存在；原来用null表示一个值不存在，现在Optional可以更优雅的表达这个概念，并且可以规避空指针异常。

容器对象，可能包含也可能不包含非null值。如果存在值，则`isPresent()`将返回true，`get()`将返回该值。

提供依赖于是否存在包含值的其他方法，例如`orElse()`（如果值不存在则返回默认值）和`ifPresent()`（如果值存在则执行代码块）。

这是一个基于价值的课程;在Optional的实例上使用身份敏感操作（包括引用相等（==），标识哈希代码或同步）可能会产生不可预测的结果，应该避免使用。

## 常用方法

- static <T> Optional<T>	of(T value)

返回具有指定的当前非null值的Optional。

注意：入参value如果为null，则会抛出空指针异常。

```java
Optional<String> optional = Optional.of("Hello World");

// java.lang.NullPointerException
String str = null;
Optional<String> optional2 = Optional.of(str);
```

- static <T> Optional<T>	empty()

返回一个空的Optional实例。

- static <T> Optional<T>	ofNullable(T value)

如果为非null，返回描述指定值的Optional；否则返回空Optional。

```java
Optional<String> optional = Optional.ofNullable("Hello World");

// 返回 Optional.empty() 对象
String str = null;
Optional<String> optional2 = Optional.ofNullable(str);
```
 
- boolean	isPresent()

如果存在值则返回true，否则返回false。

- T	get()

如果此Optional中存在值，则返回该值，否则抛出NoSuchElementException。

- void	ifPresent(Consumer<? super T> consumer)

如果存在值，则使用值调用指定的使用者，否则不执行任何操作。

感觉相当于 isPresent() 和 get() 的合并用法

```java
Optional<String> optional = Optional.ofNullable(null);
if (optional.isPresent()) {
    System.out.println(optional.get());
}

optional.ifPresent(t -> {
    System.out.println(t);
});
```

- T	orElse(T other)

如果存在则返回值，否则返回其他值。

- T	orElseGet(Supplier<? extends T> other)

返回值（如果存在），否则调用other并返回该调用的结果。

```java
Optional<String> optional = Optional.ofNullable(null);
String result = optional.orElse("默认值");
System.out.println(result);

String result2 = optional.orElseGet(() -> {
    return "Default Value";
});
System.out.println(result2);
```

- <X extends Throwable> T	orElseThrow(Supplier<? extends X> exceptionSupplier)

返回包含的值（如果存在），否则抛出由提供的供应商创建的异常。

```java
Optional<String> optional2 = Optional.ofNullable(null);
String result2 = optional2.orElseThrow(() -> new RuntimeException());
System.out.println(result2);
```
 
 ## 个人感觉
 
 最初使用时，感觉有些鸡肋；
 后来慢慢的用着用着，有一种感觉，确实比`if(obj == null)`看着要舒服，纯属个人感觉。
 
 
## 文档

[https://docs.oracle.com/javase/8/docs/api/][https://docs.oracle.com/javase/8/docs/api/]






