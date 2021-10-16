---
layout:     post
title:      java探索之native方法源码实现
subtitle:   原来一直好奇jdk源码中的native方法为什么只有声明，而没有实现内容，希望这篇文章给你一点启示。
date:       2019-02-16
author:     BY morningcat
header-img: img/20190216/post-bg-coffee.jpeg
catalog: true
tags:
    - java
---



## 背景

不晓得小伙伴们在学习`java`时有没有遇到过使用`native`关键字修饰的方法，我记得有一次好奇`java`类的基类`Object.java`里到底有些什么，于是就打开了`jdk`的源码看了一下；
不要问我jdk的源码怎么看，去哪里看，丑拒；
以下是`Object.java`的源码，可以看出基本都是`native`修饰的方法且并没有方法体；
```java
package java.lang;

public class Object {

    private static native void registerNatives();
    static {
        registerNatives();
    }

    public final native Class<?> getClass();

    public native int hashCode();

    public boolean equals(Object obj) {
        return (this == obj);
    }

    protected native Object clone() throws CloneNotSupportedException;

    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }

    public final native void notify();

    public final native void notifyAll();

    public final native void wait(long timeout) throws InterruptedException;

    public final void wait(long timeout, int nanos) throws InterruptedException {
        ···
        wait(timeout);
    }

    public final void wait() throws InterruptedException {
        wait(0);
    }

    protected void finalize() throws Throwable { }
}
```
上网查询了一下，得到一个简单的结论，`native`是`java`本地接口；··· 懵逼中···
上网搜索JNI(Java Native Interface)，可获取详细信息，此处不多言；
当时只晓得这些本地方法使用`c语言`编写的，于是一直想找到这些`native`方法的实现拿来看一看，就在不久前这个愿望实现了，下面介绍过程；


## 源码

根据CSDN中的一篇[博客](https://blog.csdn.net/kelindame/article/details/44625255)的提示，找到了下载openjdk的源码的[下载地址](http://jdk.java.net/)；
可以选择`jdk`的版本，这里选择[jdk1.8版本](http://jdk.java.net/java-se-ri/8)进行[下载](https://download.java.net/openjdk/jdk8u40/ri/openjdk-8u40-src-b25-10_feb_2015.zip);
提示：
```
RI Source Code
The source code of the RI binaries is available under the GPLv2 in a single zip file (md5) 121 MB.
```
找到如上一句话，点击`zip file`即可下载；

下载完成后进行解压；
进入 openjdk -> jdk -> src 目录下；
目录下有7个目录，每一个目录下基本都有一个`native`目录（有些没有），其中有6个目录名都是操作系统的种类名称，还有一个是`share`目录；
含义就是：
`share`目录的`native`下是 平台无关的实现；
其他都是各个系统的的定制实现，
这大概就是`java`语言可以跨平台使用的一个必要的步骤吧，需要为每一类操作系统定制化一些实现。

以下为`object.c`源码
```c
#include <stdio.h>
#include <signal.h>
#include <limits.h>

#include "jni.h"
#include "jni_util.h"
#include "jvm.h"

#include "java_lang_Object.h"

static JNINativeMethod methods[] = {
    {"hashCode",    "()I",                    (void *)&JVM_IHashCode},
    {"wait",        "(J)V",                   (void *)&JVM_MonitorWait},
    {"notify",      "()V",                    (void *)&JVM_MonitorNotify},
    {"notifyAll",   "()V",                    (void *)&JVM_MonitorNotifyAll},
    {"clone",       "()Ljava/lang/Object;",   (void *)&JVM_Clone},
};

JNIEXPORT void JNICALL
Java_java_lang_Object_registerNatives(JNIEnv *env, jclass cls)
{
    (*env)->RegisterNatives(env, cls,
                            methods, sizeof(methods)/sizeof(methods[0]));
}

JNIEXPORT jclass JNICALL
Java_java_lang_Object_getClass(JNIEnv *env, jobject this)
{
    if (this == NULL) {
        JNU_ThrowNullPointerException(env, NULL);
        return 0;
    } else {
        return (*env)->GetObjectClass(env, this);
    }
}
```

## 其他下载地址

- [jdk7](https://download.java.net/openjdk/jdk7u75/ri/openjdk-7u75-src-b13-18_dec_2014.zip)
- [jdk8](https://download.java.net/openjdk/jdk8u40/ri/openjdk-8u40-src-b25-10_feb_2015.zip)
- [jdk9](https://download.java.net/openjdk/jdk9/ri/openjdk-9_src.zip)
- [jdk10](https://download.java.net/openjdk/jdk10/ri/openjdk-10_src.zip)
- [jdk11](https://download.java.net/openjdk/jdk11/ri/openjdk-11+28_src.zip)

完成，睡觉。
