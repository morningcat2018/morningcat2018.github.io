---
layout:     post
title:      使用IDEA查看java文件编译后的字节码内容
subtitle:   字节码
date:       2019-03-08
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - java
---


1. 首先编写一个java类 `StringDemo1.java`
```java
public class StringDemo1 {

    public static void main(String[] args) {
        String str1 = "aaa" + "bbb";
        System.out.println(str1);


        String str2 = "ccc";
        str2 += "ddd";
        System.out.println(str2);
    }
}
```
2. 经过编译后，生成 `StringDemo1.class`文件
使用IDEA查看编译之后的文件内容：
```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package jdk.java.lang.string;

public class StringDemo1 {
    public StringDemo1() {
    }

    public static void main(String[] args) {
        String str1 = "aaabbb";
        System.out.println(str1);
        String str2 = "ccc";
        str2 = str2 + "ddd";
        System.out.println(str2);
    }
}
```
可以看出内容已经是经过IDEA反编译之后的了，并不能看到字节码内容。

3. 使用`jclasslib`反编译工具
在IDEA中搜索插件`jclasslib bytecode viewer `，然后进行安装，重启IDEA。
再次打开`StringDemo1.java`文件，在IDEA菜单栏上打开`View -> Show Bytecode`，即可以看到字节码文件内容:
```java
// class version 52.0 (52)
// access flags 0x21
public class jdk/java/lang/string/StringDemo1 {

  // compiled from: StringDemo1.java

  // access flags 0x1
  public <init>()V
   L0
    LINENUMBER 8 L0
    ALOAD 0
    INVOKESPECIAL java/lang/Object.<init> ()V
    RETURN
   L1
    LOCALVARIABLE this Ljdk/java/lang/string/StringDemo1; L0 L1 0
    MAXSTACK = 1
    MAXLOCALS = 1

  // access flags 0x9
  public static main([Ljava/lang/String;)V
   L0
    LINENUMBER 11 L0
    LDC "aaabbb"
    ASTORE 1
   L1
    LINENUMBER 12 L1
    GETSTATIC java/lang/System.out : Ljava/io/PrintStream;
    ALOAD 1
    INVOKEVIRTUAL java/io/PrintStream.println (Ljava/lang/String;)V
   L2
    LINENUMBER 15 L2
    LDC "ccc"
    ASTORE 2
   L3
    LINENUMBER 16 L3
    NEW java/lang/StringBuilder
    DUP
    INVOKESPECIAL java/lang/StringBuilder.<init> ()V
    ALOAD 2
    INVOKEVIRTUAL java/lang/StringBuilder.append (Ljava/lang/String;)Ljava/lang/StringBuilder;
    LDC "ddd"
    INVOKEVIRTUAL java/lang/StringBuilder.append (Ljava/lang/String;)Ljava/lang/StringBuilder;
    INVOKEVIRTUAL java/lang/StringBuilder.toString ()Ljava/lang/String;
    ASTORE 2
   L4
    LINENUMBER 17 L4
    GETSTATIC java/lang/System.out : Ljava/io/PrintStream;
    ALOAD 2
    INVOKEVIRTUAL java/io/PrintStream.println (Ljava/lang/String;)V
   L5
    LINENUMBER 18 L5
    RETURN
   L6
    LOCALVARIABLE args [Ljava/lang/String; L0 L6 0
    LOCALVARIABLE str1 Ljava/lang/String; L1 L6 1
    LOCALVARIABLE str2 Ljava/lang/String; L3 L6 2
    MAXSTACK = 2
    MAXLOCALS = 3
}
```
以上编译内容是我在`JDK1.8u202`环境下进行编译的，所以开头才会有`class version 52.0 (52)`的标识。

至于字节码怎么读，就需要继续学习；
[Java虚拟机规范](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html)中有关于`Java虚拟机指令集`的相关资料，可以学习研究一下。

随后还可以学习一下`ASM技术`([Java字节码操控框架](https://asm.ow2.io/))；

4. JDK版本映射

| JDK版本 | class版本 |
|--|--|
| J2SE 8 | 52 |
| J2SE 7 |  51 |
| J2SE 6.0 | 50 |
| J2SE 5.0 |  49
| JDK 1.4 |  48
| JDK 1.3 |  47
| JDK 1.2 |  46
| JDK 1.1 |  45

以上，继续努力