---
layout:     post
title:      DCL 为什么还需要 volatile
subtitle:   
date:       2020-05-08
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - java
---

## DCL

Double-checked locking [双重检查锁定模式](https://zh.wikipedia.org/wiki/%E5%8F%8C%E9%87%8D%E6%A3%80%E6%9F%A5%E9%94%81%E5%AE%9A%E6%A8%A1%E5%BC%8F)

双重检查锁定模式首先验证锁定条件(第一次检查)，只有通过锁定条件验证才真正的进行加锁逻辑并再次验证条件(第二次检查)。

为多线程环境中的单例模式实现“惰性初始化”。


```java
// Broken multithreaded version
// "Double-Checked Locking" idiom
class Foo {
    private Helper helper = null;
    public Helper getHelper() {
        if (helper == null) {
            synchronized(this) {
                if (helper == null) {
                    helper = new Helper();
                }
            }
        }
        return helper;
    }

    // other functions and members...
}
```

直觉上，上述写法看起来是没有问题的。

但是有一种极地概率的事件可能发生；

## 为什么还需要 volatile

```java
public class Foo {
    Node node = new Node(12);
}

class Node {

    int v;

    public Node() {
    }

    public Node(int v) {
        this.v = v;
    }
}
```

> javap -c jdk.java.util.concurrent.dcl.Foo

```
public class jdk.java.util.concurrent.dcl.Foo {
  jdk.java.util.concurrent.dcl.Node node;

  public jdk.java.util.concurrent.dcl.Foo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: aload_0
       5: new           #2                  // class jdk/java/util/concurrent/dcl/Node
       8: dup
       9: bipush        12
      11: invokespecial #3                  // Method jdk/java/util/concurrent/dcl/Node."<init>":(I)V
      14: putfield      #4                  // Field node:Ljdk/java/util/concurrent/dcl/Node;
      17: return
}
```

new 出一个对象，简要的概括可以分为三步：
1.在堆上分配一块内存区域；                           // 5: new           #2
2.对象数据初始化；                                  // 9: bipush        12
3.将符号引用(上诉代码中的 node )指向堆内的实际内存地址   // 14: putfield      #4

但如果发生在 9 和 14 之间发生了指令重排序，就会导致 node 先指向了一个内存地址，然后再进行初始化的不合理顺序；
在超大并发情况下，线程a得到锁，进入锁内部执行 new Node() 的操作，然而此时发生上诉所说问题，此时线程b 获取对象，拿到对象发现不为null，返回后直接开始使用了，但其实该对象并没有初始化完成，并不是一个实例化完成的对象。

维基百科上是这样描述的：
1. 线程A发现变量没有被初始化, 然后它获取锁并开始变量的初始化。
2. 由于某些编程语言的语义，编译器生成的代码允许在线程A执行完变量的初始化之前，更新变量并将其指向部分初始化的对象。
3. 线程B发现共享变量已经被初始化，并返回变量。由于线程B确信变量已被初始化，它没有获取锁。如果在A完成初始化之前共享变量对B可见（这是由于A没有完成初始化或者因为一些初始化的值还没有覆盖B使用的内存(缓存一致性)），程序很可能会崩溃。

在J2SE 1.4 或更早的版本中使用双重检查锁有潜在的危险，有时会正常工作：区分正确实现和有小问题的实现是很困难的。取决于编译器，线程的调度和其他并发系统活动，不正确的实现双重检查锁导致的异常结果可能会间歇性出现。重现异常是十分困难的。

在J2SE 5.0中，这一问题被修正了。volatile关键字保证多个线程可以正确处理单件实例。

```java
// Works with acquire/release semantics for volatile
// Broken under Java 1.4 and earlier semantics for volatile
class Foo {
    private volatile Helper helper = null;
    public Helper getHelper() {
        Helper result = helper;
        if (result == null) {
            synchronized(this) {
                result = helper;
                if (result == null) {
                    helper = result = new Helper();
                }
            }
        }
        return result;
    }

    // other functions and members...
}
```
注意局部变量result的使用看起来是不必要的。对于某些版本的Java虚拟机，这会使代码提速25%，而对其他的版本则无关痛痒。

## 替代方案

如果helper对象是静态的(每个类只有一个), 可以使用双重检查锁的替代模式惰性初始化模式。

```java
// Correct lazy initialization in Java
@ThreadSafe
class Foo {
    private static class HelperHolder {
       public static Helper helper = new Helper();
    }

    public static Helper getHelper() {
        return HelperHolder.helper;
    }
}
```

这是因为内部类直到他们被引用时才会加载。

## 替代方案2

Java 5中的final语义可以不使用volatile关键字实现安全的创建对象：

```java
public class FinalWrapper<T> {
    public final T value;
    public FinalWrapper(T value) {
        this.value = value;
    }
}

public class Foo {
   private FinalWrapper<Helper> helperWrapper = null;

   public Helper getHelper() {
      FinalWrapper<Helper> wrapper = helperWrapper;

      if (wrapper == null) {
          synchronized(this) {
              if (helperWrapper == null) {
                  helperWrapper = new FinalWrapper<Helper>(new Helper());
              }
              wrapper = helperWrapper;
          }
      }
      return wrapper.value;
   }
}
```

为了正确性，局部变量wrapper是必须的。这一实现的性能不一定比使用volatile的性能更高。



