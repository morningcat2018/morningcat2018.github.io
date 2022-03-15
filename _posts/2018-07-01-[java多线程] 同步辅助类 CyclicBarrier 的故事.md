---
layout:     post
title:      同步辅助类 CyclicBarrier 的故事
subtitle:   根据 Jdk 提供的Java多线程同步辅助类 CyclicBarrier 的用法 编写的一个小场景，小故事
date:       2019-03-15
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - java多线程
---
## 同步辅助类 CyclicBarrier 的故事

```java
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Random;
import java.util.concurrent.CyclicBarrier;

/**
 * @describe: CyclicBarrier 的故事
 * @author: morningcat.zhang
 * @date: 2019/3/1 2:00 PM
 */
public class CyclicBarrierTest {
    private static final int THREAD_COUNT_NUM = 8;

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(THREAD_COUNT_NUM, () -> System.out.println(nowTime() + "---集合点---"));

        for (int i = 0; i < THREAD_COUNT_NUM; i++) {
            final int num = i + 1;
            Runnable runnable = () -> {
                // 完成相关业务 ...
                try {
                    Thread.sleep(new Random().nextInt(10000));
                    System.out.println(nowTime() + num + "号人员已就位");
                    cyclicBarrier.await();

                    System.out.println(nowTime() + num + "号人员开始操练");
                    Thread.sleep(new Random().nextInt(10000));
                    System.out.println(nowTime() + num + "号人员听到集合口哨立即跑来集合点");
                    cyclicBarrier.await();

                    Thread.sleep(new Random().nextInt(10000));
                    System.out.println(nowTime() + num + "号人员到达食堂");
                    cyclicBarrier.await();

                } catch (Exception e) {
                    e.printStackTrace();
                }
            };

            new Thread(runnable).start();
        }

        System.out.println("---新的一天---");
    }

    public static String nowTime() {
        return LocalDateTime.now().format(DateTimeFormatter.ofPattern("[YYYY-MM-dd HH:mm:ss.SSS] --- "));
    }

}

```

## 语法简介

一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier 在释放等待线程后可以重用，所以称它为循环 的 barrier。

CyclicBarrier 支持一个可选的 Runnable 命令，在一组线程中的最后一个线程到达之后（但在释放所有线程之前），该命令只在每个屏障点运行一次。若在继续所有参与线程之前更新共享状态，此屏障操作 很有用。

### 构造方法摘要
CyclicBarrier(int parties) 
          创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，但它不会在启动 barrier 时执行预定义的操作。
          
CyclicBarrier(int parties, Runnable barrierAction) 
          创建一个新的 CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，并在启动 barrier 时执行给定的屏障操作，该操作由最后一个进入 barrier 的线程执行。
 
### 方法摘要
 int	await() 
          在所有参与者都已经在此 barrier 上调用 await 方法之前，将一直等待。
 int	await(long timeout, TimeUnit unit) 
          在所有参与者都已经在此屏障上调用 await 方法之前将一直等待,或者超出了指定的等待时间。
 int	getNumberWaiting() 
          返回当前在屏障处等待的参与者数目。
 int	getParties() 
          返回要求启动此 barrier 的参与者数目。
 boolean	isBroken() 
          查询此屏障是否处于损坏状态。
 void	reset() 
          将屏障重置为其初始状态。
 