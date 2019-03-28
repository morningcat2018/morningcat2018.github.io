---
layout:     post
title:      信号量Semaphore的故事
subtitle:   根据 Jdk 提供的Java多线程控制类 Semaphore 的用法 编写的一个小场景，小故事
date:       2019-02-14
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - java多线程
---


## 信号量Semaphore 的故事
```java
import org.junit.Test;
import java.util.concurrent.Semaphore;

/**
 * @describe: 信号量 Semaphore 的使用
 * @author: morningcat.zhang
 * @date: 2019/2/14 10:05 PM
 */
public class SemaphoreTest {

    /**
     * 第1天
     */
    @Test
    public void test1() {

        /**
         * 网吧A 有40个位置 （每小时2元）
         */
        Semaphore semaphoreA = new Semaphore(40);
        try {
            // 小明来到网吧想上网，去咨询网管有没有位置了
            // 若没有位置了，则坐在旁边等待一会（线程阻塞）
            semaphoreA.acquire();

            // 等到了有人下机，轮到我上机了
            // （开始自己的业务操作）。。。

        } catch (InterruptedException e) {
            // acquire() 方法是有中断线程的情况发现，则不再等待
            // 如果你同学说我们去滑冰吧，不在网吧上网了，你直接去滑冰去了
            e.printStackTrace();
        } finally {
            // 上网时间到了，离开网吧的位置（线程回收）
            semaphoreA.release();
        }

    }

    /**
     * 第2天
     */
    @Test
    public void test2() {

        /**
         * 网吧B 只有30个位置 （每小时3元）
         * <p/>
         * 话说为什么不去网吧A 上网了呢，网吧A 价格便宜位置又多，原因是：
         * 小明发现，网吧A 的网管不公平，明明是我先来排队的，但是有人下机时网管没有先叫早就在等待的我，
         * 而是随便喊了一个刚来没等待多久的妹子去先上机了，等了很久的我实在很生气（Semaphore 默认是非公平锁）
         * <p/>
         * 而网吧B 就不一样了，当位置满了的时候，后来每来一个人都会给一个编号，然后有人下机时顺序叫号（公平锁）
         * 当然网吧B 因为增加了一个叫号系统 网管当然就要辛苦一些 (设置成公平锁会损耗一部分性能)
         */
        Semaphore semaphoreB = new Semaphore(30, true);
        try {
            // 这次小明来到网吧上网，还带了自己的2个好朋友一起，所以这次他需要3个位置
            // 若没有3个位置，他们则一起坐在旁边等待一会（线程阻塞）
            semaphoreB.acquire(3);

            // 等到了有3个空位置，就可以一起快乐的玩耍了
            // （开始自己的业务操作）。。。

        } catch (InterruptedException e) {
            // acquire() 方法是有中断线程的情况发现，则不再等待
            // 如果你同学说我们去滑冰吧，不在网吧上网了，你直接去滑冰去了
            e.printStackTrace();
        } finally {
            // 上网时间到了，3人同时离开网吧，释放三个位置（线程回收）
            semaphoreB.release(3);
        }

    }


    /**
     * 第8天
     */
    @Test
    public void test8() {

        /**
         * 网吧B 只有30个位置 （每小时3元）
         * <p/>
         * 话说小明已经好久没有上网了，他决定这次一定要玩个够，谁喊我去做其他事都不会去的
         */
        Semaphore semaphoreB = new Semaphore(30, true);

        semaphoreB.acquireUninterruptibly();
        // 等了好久，同学喊我去打牌没有去，去滑冰也没有去，一直在等着上网，这次要玩个够
        //（开始自己的业务操作）。。。
        semaphoreB.release();
    }


    /**
     * 第N天
     */
    @Test
    public void testN() {

        /**
         * 网吧C 只有50个位置 （每小时2元）
         * <p/>
         * 小亮是一个学霸，偶尔也会来网吧娱乐一下
         */
        Semaphore semaphoreC = new Semaphore(50);

        // 问一下网管有没有位置了
        boolean leisured = semaphoreC.tryAcquire();
        if (leisured) {
            // 有空闲位置，上网娱乐一下
            //（开始自己的业务操作）。。。
        } else {
            // 网吧没有位置了，看来老天都想让我回家学习，算了回家吧
        }
    }

}

```

## 语法简介
### 构造方法摘要
Semaphore(int permits) 
          创建具有给定的许可数和非公平的公平设置的 Semaphore。
Semaphore(int permits, boolean fair) 
          创建具有给定的许可数和给定的公平设置的 Semaphore。
 
### 方法摘要
 void	acquire() 
          从此信号量获取一个许可，在提供一个许可前一直将线程阻塞，否则线程被中断。
 void	acquire(int permits) 
          从此信号量获取给定数目的许可，在提供这些许可前一直将线程阻塞，或者线程已被中断。
 void	acquireUninterruptibly() 
          从此信号量中获取许可，在有可用的许可前将其阻塞。
 void	acquireUninterruptibly(int permits) 
          从此信号量获取给定数目的许可，在提供这些许可前一直将线程阻塞。
 int	availablePermits() 
          返回此信号量中当前可用的许可数。
 int	drainPermits() 
          获取并返回立即可用的所有许可。
protected  Collection<Thread>	getQueuedThreads() 
          返回一个 collection，包含可能等待获取的线程。
 int	getQueueLength() 
          返回正在等待获取的线程的估计数目。
 boolean	hasQueuedThreads() 
          查询是否有线程正在等待获取。
 boolean	isFair() 
          如果此信号量的公平设置为 true，则返回 true。
protected  void	reducePermits(int reduction) 
          根据指定的缩减量减小可用许可的数目。
 void	release() 
          释放一个许可，将其返回给信号量。
 void	release(int permits) 
          释放给定数目的许可，将其返回到信号量。
 String	toString() 
          返回标识此信号量的字符串，以及信号量的状态。
 boolean	tryAcquire() 
          仅在调用时此信号量存在一个可用许可，才从信号量获取许可。
 boolean	tryAcquire(int permits) 
          仅在调用时此信号量中有给定数目的许可时，才从此信号量中获取这些许可。
 boolean	tryAcquire(int permits, long timeout, TimeUnit unit) 
          如果在给定的等待时间内此信号量有可用的所有许可，并且当前线程未被中断，则从此信号量获取给定数目的许可。
 boolean	tryAcquire(long timeout, TimeUnit unit) 
          如果在给定的等待时间内，此信号量有可用的许可并且当前线程未被中断，则从此信号量获取一个许可。
 
---

### 构造方法 及常用方法
```java
Semaphore(int permits)
Semaphore(int permits, boolean fair)

public void acquire()
public void acquireUninterruptibly()
public boolean tryAcquire()
public boolean tryAcquire(long timeout, TimeUnit unit)
public void release()

public void acquire(int permits)
public void acquireUninterruptibly(int permits)
public boolean tryAcquire(int permits)
public boolean tryAcquire(int permits, long timeout, TimeUnit unit)
public void release(int permits)
```

### 有待学习的方法
```java
public int availablePermits()
public int drainPermits()
protected void reducePermits(int reduction)
public boolean isFair()

public final boolean hasQueuedThreads()
public final int getQueueLength()
protected Collection<Thread> getQueuedThreads()
```

## 参考资料

https://zhuanlan.zhihu.com/p/28769523

https://www.jianshu.com/p/d8eeb31bee5c

https://www.jianshu.com/p/0090341c6b80