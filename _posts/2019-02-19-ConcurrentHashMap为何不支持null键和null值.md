---
layout:     post
title:      ConcurrentHashMap为何不支持null键和null值
subtitle:   java集合类 ConcurrentHashMap 的一些认识
date:       2019-02-19
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - java多线程
---


## 背景

最近在梳理总结[《集合 - 常用Map之间区别》](https://my.oschina.net/mengzhang6/blog/1786778)，
其中有一点就是 HashMap 是支持null键和null值，而 ConcurrentHashMap 是不支持的；

后来查看了一下jdk源码，证明了确实是这样的。

HashMap.java 部分源码
```
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

ConcurrentHashMap.java 部分源码
```
    /** Implementation for put and putIfAbsent */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        ......
    }
```

## 知其然不知其所以然

后来，当有人问起我为什么ConcurrentHashMap不支持null key value时，我却只能说源码里就那么写的，却不知为何！


## 知其所以然

上网google了一下，发现一片很不错的文章[关于ConcurrentHashMap为什么不能put null](https://laiqitech.com/125/),正好能解释我的疑惑。

里面提到：
```
ConcurrentHashMap不能put null 是因为 无法分辨是key没找到的null还是有key值为null，这在多线程里面是模糊不清的，所以压根就不让put null。

```

里面重点提到一个大师Doug Lea，据说是ConcurrentHashMap的作者，赶紧查看了源码：
```
 /**
 * ...
 * Java Collections Framework</a>.
 *
 * @since 1.5
 * @author Doug Lea
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 */
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
    private static final long serialVersionUID = 7249069246763182397L;
    ...
}
```

原来整个java.util.concurrent大多都是[Lea大师](http://g.oswego.edu/)的作品，值得细细品味。

## 相关资料

- [并发编程网 - ifeve.com](http://ifeve.com/doug-lea/)
- [你应当知道的Java牛人 v2.0](http://www.importnew.com/5575.html)
- [Doug Lea 大神关于Java 7引入的他写的Fork/Join框架的论文](https://juejin.im/entry/5a027e2bf265da43247fdef7)

## [开源中国-晨猫](https://my.oschina.net/mengzhang6/blog/2961834)
