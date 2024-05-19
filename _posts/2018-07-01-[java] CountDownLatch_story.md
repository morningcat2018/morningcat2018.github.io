---
layout:     post
title:      多线程控制工具类 CountDownLatch 的故事
subtitle:   根据 Jdk 提供的Java多线程控制类 CountDownLatch 的用法 编写的一个小场景，小故事
date:       2019-03-03
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - java
---

## 多线程控制工具类 CountDownLatch 的故事

```java
package jdk.java.util.concurrent;

import org.junit.Test;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Random;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

/**
 * @describe: CountDownLatch 的故事
 * @author: morningcat.zhang
 * @date: 2019/3/1 1:22 PM
 */
public class CountDownLatchTest {

    private String nowTime() {
        return LocalDateTime.now().format(DateTimeFormatter.ofPattern("[YYYY-MM-dd HH:mm:ss.SSS] --- "));
    }

    @Test
    public void test1() throws InterruptedException {
        // 一连 有8个成员（每天起床给十秒时间）
        CountDownLatch countDownLatch = new CountDownLatch(8);
        for (int i = 0; i < 8; i++) {
            int num = i + 1;
            new Thread(() -> doWork(countDownLatch, num)).start();
        }

        // 在此等待
        System.out.println(nowTime() + "寝室门口等待集合--->");
        countDownLatch.await();

        // 全部到达
        System.out.println(nowTime() + "全部到达，出发去操练");

    }

    private void doWork(CountDownLatch countDownLatch, int num) {
        try {
            // 完成相关业务
            Thread.sleep(new Random().nextInt(10000));
            System.out.println(nowTime() + num + "号人员已就位");

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            countDownLatch.countDown();
        }
    }

    @Test
    public void test2() throws InterruptedException {
        // 二连 有8个成员（每天起床给十秒时间，且过时不候）
        CountDownLatch countDownLatch = new CountDownLatch(8);
        for (int i = 0; i < 8; i++) {
            int num = i + 1;
            new Thread(() -> doWork2(countDownLatch, num)).start();
        }

        // 在此等待
        System.out.println(nowTime() + "寝室门口等待集合(最久等10s)--->");
        countDownLatch.await(10, TimeUnit.SECONDS);

        // 全部到达
        System.out.println(nowTime() + "过时不候，出发去操练");
    }

    private void doWork2(CountDownLatch countDownLatch, int num) {
        if (num == 4) {
            // 4号今天请假了
            return;
        }
        try {
            // 完成相关业务
            Thread.sleep(new Random().nextInt(5000));
            System.out.println(nowTime() + num + "号人员已就位");

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            countDownLatch.countDown();
        }
    }
}

```

## 语法简介

用给定的计数 初始化 CountDownLatch。由于调用了 countDown() 方法，所以在当前计数到达零之前，await 方法会一直受阻塞。之后，会释放所有等待的线程，await 的所有后续调用都将立即返回。

CountDownLatch 是一个通用同步工具，它有很多用途。将计数 1 初始化的 CountDownLatch 用作一个简单的开/关锁存器，或入口：在通过调用 countDown() 的线程打开入口前，所有调用 await 的线程都一直在入口处等待。用 N 初始化的 CountDownLatch 可以使一个线程在 N 个线程完成某项操作之前一直等待，或者使其在某项操作完成 N 次之前一直等待。

CountDownLatch 的一个有用特性是，它不要求调用 countDown 方法的线程等到计数到达零时才继续，而在所有线程都能通过之前，它只是阻止任何线程继续通过一个 await。

### 构造方法摘要
CountDownLatch(int count) 
          构造一个用给定计数初始化的 CountDownLatch。
 
### 方法摘要
 void	await() 
          使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断。
 boolean	await(long timeout, TimeUnit unit) 
          使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断或超出了指定的等待时间。
 void	countDown() 
          递减锁存器的计数，如果计数到达零，则释放所有等待的线程。
 long	getCount() 
          返回当前计数。
 String	toString() 
          返回标识此锁存器及其状态的字符串。

## 附录
jdk版本：jdk1.8u202