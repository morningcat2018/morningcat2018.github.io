---
layout:     post
title:      关于 IntegerCache 的一些认识
subtitle:   与君共勉
date:       2019-03-04
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - jdk源码
---


## 关于 IntegerCache 的一些认识

### 从一个小问题开始入手

下面程序的输出结果是什么？
```java
    @Test
    public void test1() {

        Integer num1 = 100;
        Integer num2 = 100;
        Integer num3 = 200;
        Integer num4 = 200;

        System.out.println(num1 == num2);
        System.out.println(num3 == num4);
    }
```

答案是：`无可奉告，自行验证`
笔者JDK版本为JDK1.8u202

### 笔者浅陋分析

`非new生成的Integer变量`指向的是`调用Integer.valueOf()方法`返回的对象；
Integer num1 = 100; 相当于 
Integer num1 = Integer.valueOf(100);

打开 valueOf 方法的源码进行分析：
```java
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```
从源码可以看出，当传入的整数在一定范围内，则从 IntegerCache的缓存中返回对象；

而IntegerCache为Integer类的内部类；
IntegerCache.low 为 -128 ；
IntegerCache.high 的值默认为 127 ，且可以通过 java.lang.Integer.IntegerCache.high 进行修改，
不过这个指定值必须比 127 大，否则IntegerCache.high 的值依然是 127。
```java
    private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```

### 思考题

题目1：
```java
    @Test
    public void test1() {
        Integer num1 = 100;
        Integer num2 = 100;
        Integer num3 = Integer.valueOf(100);
        Integer num4 = Integer.valueOf(100);

        Integer num5 = new Integer(100);
        Integer num6 = new Integer(100);


        System.out.println(num1 == num2);
        System.out.println(num3 == num4);
        System.out.println(num5 == num6);

        System.out.println(num1 == num3);
        System.out.println(num1 == num5);
        System.out.println(num3 == num5);
    }
```


题目2：
```java
    @Test
    public void test2() {
        Integer num1 = 200;
        Integer num2 = 200;
        Integer num3 = Integer.valueOf(200);
        Integer num4 = Integer.valueOf(200);

        Integer num5 = new Integer(200);
        Integer num6 = new Integer(200);


        System.out.println(num1 == num2);
        System.out.println(num3 == num4);
        System.out.println(num5 == num6);

        System.out.println(num1 == num3);
        System.out.println(num1 == num5);
        System.out.println(num3 == num5);
    }
```

题目3：
```
在不修改代码的情况下，如何使题目2的答案和题目3的答案一样。

```

### 最后

--|---|.-.|-.|..|-.|--.|-.-.|.-|-|.-..|..|-.-|.|--.|---|..-|--..|..|
