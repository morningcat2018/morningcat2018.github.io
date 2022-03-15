---
layout:     post
title:      关于 ScheduledThreadPoolExecutor 的一些资料汇总
subtitle:   及个人理解
date:       2019-04-19
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_656.jpg
catalog: true
tags:
    - java多线程
---

关于`ScheduledThreadPoolExecutor`的一些资料汇总及个人理解

相关资料：
- [关于 ThreadPoolExecutor 的一些资料汇总及个人认识](https://blog.csdn.net/u013837825/article/details/89384871)

## 1.资料汇总

```java

/**
 * @since 1.5
 * @author Doug Lea
 */
public class ScheduledThreadPoolExecutor
        extends ThreadPoolExecutor
        implements ScheduledExecutorService {
    // ...
}
```

- 构造方法

```
ScheduledThreadPoolExecutor(int corePoolSize) 
          使用给定核心池大小创建一个新 ScheduledThreadPoolExecutor。
          
ScheduledThreadPoolExecutor(int corePoolSize, RejectedExecutionHandler handler) 
          使用给定初始参数创建一个新 ScheduledThreadPoolExecutor。
          
ScheduledThreadPoolExecutor(int corePoolSize, ThreadFactory threadFactory) 
          使用给定的初始参数创建一个新 ScheduledThreadPoolExecutor。
          
ScheduledThreadPoolExecutor(int corePoolSize, ThreadFactory threadFactory, RejectedExecutionHandler handler) 
          使用给定初始参数创建一个新 ScheduledThreadPoolExecutor。
```

- 参数说明

```
corePoolSize - 池中所保存的线程数（包括空闲线程）
threadFactory - 执行程序创建新线程时使用的工厂
handler - 由于超出线程范围和队列容量而使执行被阻塞时所使用的处理程序
```

- 抛出异常

```
IllegalArgumentException - 如果 corePoolSize < 0
NullPointerException - 如果处理程序为 null
```

- 方法摘要

```

 protected <V> RunnableScheduledFuture<V> decorateTask(Callable<V> callable, RunnableScheduledFuture<V> task) 
          修改或替换用于执行 callable 的任务。
 
 protected <V> RunnableScheduledFuture<V> decorateTask(Runnable runnable, RunnableScheduledFuture<V> task) 
          修改或替换用于执行 runnable 的任务。
 
 void	execute(Runnable command) 
          使用所要求的零延迟执行命令。
 
 boolean	getContinueExistingPeriodicTasksAfterShutdownPolicy() 
          获取有关在此执行程序已 shutdown 的情况下、是否继续执行现有定期任务的策略。
 
 boolean	getExecuteExistingDelayedTasksAfterShutdownPolicy() 
          获取有关在此执行程序已 shutdown 的情况下是否继续执行现有延迟任务的策略。
 
 BlockingQueue<Runnable>	getQueue() 
          返回此执行程序使用的任务队列。
 
 boolean	remove(Runnable task) 
          从执行程序的内部队列中移除此任务（如果存在），从而如果尚未开始，则其不再运行。
 
 <V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit) 
          创建并执行在给定延迟后启用的 ScheduledFuture。
 
 ScheduledFuture<?>	schedule(Runnable command, long delay, TimeUnit unit) 
          创建并执行在给定延迟后启用的一次性操作。
 
 ScheduledFuture<?>	scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit) 
          创建并执行一个在给定初始延迟后首次启用的定期操作，后续操作具有给定的周期；也就是将在 initialDelay 后开始执行，然后在 initialDelay+period 后执行，接着在 initialDelay + 2 * period 后执行，依此类推。
 
 ScheduledFuture<?>	scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit) 
          创建并执行一个在给定初始延迟后首次启用的定期操作，随后，在每一次执行终止和下一次执行开始之间都存在给定的延迟。
 
 void	setContinueExistingPeriodicTasksAfterShutdownPolicy(boolean value) 
          设置有关在此执行程序已 shutdown 的情况下是否继续执行现有定期任务的策略。
 
 void	setExecuteExistingDelayedTasksAfterShutdownPolicy(boolean value) 
          设置有关在此执行程序已 shutdown 的情况下是否继续执行现有延迟任务的策略。
 
 void	shutdown() 
          在以前已提交任务的执行中发起一个有序的关闭，但是不接受新任务。
 
 List<Runnable>	shutdownNow() 
          尝试停止所有正在执行的任务、暂停等待任务的处理，并返回等待执行的任务列表。
 
 <T> Future<T> submit(Callable<T> task) 
          提交一个返回值的任务用于执行，返回一个表示任务的未决结果的 Future。
 
 Future<?>	submit(Runnable task) 
          提交一个 Runnable 任务用于执行，并返回一个表示该任务的 Future。
 
 <T> Future<T> submit(Runnable task, T result) 
          提交一个 Runnable 任务用于执行，并返回一个表示该任务的 Future。
```


- 资料

```$xslt
ThreadPoolExecutor，它可另行安排在给定的延迟后运行命令，或者定期执行命令。需要多个辅助线程时，或者要求 ThreadPoolExecutor 具有额外的灵活性或功能时，此类要优于 Timer。

一旦启用已延迟的任务就执行它，但是有关何时启用，启用后何时执行则没有任何实时保证。按照提交的先进先出 (FIFO) 顺序来启用那些被安排在同一执行时间的任务。

虽然此类继承自 ThreadPoolExecutor，但是几个继承的调整方法对此类并无作用。特别是，因为它作为一个使用 corePoolSize 线程和一个无界队列的固定大小的池，所以调整 maximumPoolSize 没有什么效果。


```

## 2.个人理解

使用`ScheduledThreadPoolExecutor`代替`java.util.Timer`，完成单JVM上的定时任务操作；

schedule(Runnable command, long delay, TimeUnit unit) 方法只能做给定时间的延迟任务，且仅执行一次；

```$xslt
scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit) 
和
scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit) 
两个方法都能完成定时任务的循环执行的功能；
区别是：
scheduleAtFixedRate -> 每次执行的开始时刻相差 : period * TimeUnit
scheduleWithFixedDelay -> 每次执行的开始时刻相差 : period * TimeUnit + run()方法执行时间
```



## 3.个人实践


```java
package collection.queue.thread_pool.schedule;

import org.junit.Test;

import java.io.IOException;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.concurrent.ScheduledThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * @describe: 类描述信息
 * @author: morningcat.zhang
 * @date: 2019/4/19 3:15 PM
 */
public class ScheduledThreadPoolExecutorTest {

    @Test
    public void test1() {
        ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(10, r -> new Thread(r, "test1"));

        printTime();

        executor.execute(() -> {
            printTime();
        });

        printTime();
    }

    @Test
    public void test2() throws IOException {
        ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(10, r -> new Thread(r, "test2"));

        printTime();

        // 指定延迟后执行一次
        executor.schedule(() -> printTime(), 3, TimeUnit.SECONDS);

        printTime();

        System.in.read();

    }

    @Test
    public void test3() throws IOException {
        ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(10, r -> new Thread(r, "test3"));

        printTime();

        // 指定延迟后执行，然后每隔 n 循环执行
        // 每次执行的开始时刻相差 : period * TimeUnit
        executor.scheduleAtFixedRate(() -> printTime(), 5, 3, TimeUnit.SECONDS);

        printTime();

        System.in.read();

    }

    @Test
    public void test4() throws IOException {
        ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(10, r -> new Thread(r, "test4"));

        printTime();

        // 指定延迟后执行，执行完成后每隔 n 循环执行
        // 每次执行的开始时刻相差 : period * TimeUnit + run()方法执行时间
        executor.scheduleWithFixedDelay(() -> printTime(), 5, 3, TimeUnit.SECONDS);

        printTime();

        System.in.read();

    }

    private static void printTime() {
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " ---> " + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss.SSS ")));
    }
}



```

test3 执行结果

```$xslt
    
main ---> 2019-04-19 15:48:07.907
main ---> 2019-04-19 15:48:08.420
test3 ---> 2019-04-19 15:48:13.426
test3 ---> 2019-04-19 15:48:16.418
test3 ---> 2019-04-19 15:48:19.423
test3 ---> 2019-04-19 15:48:22.426
test3 ---> 2019-04-19 15:48:25.424
test3 ---> 2019-04-19 15:48:28.422
test3 ---> 2019-04-19 15:48:31.419
test3 ---> 2019-04-19 15:48:34.421
test3 ---> 2019-04-19 15:48:37.423
     
```

test4 执行结果

```$xslt
main ---> 2019-04-19 15:49:14.265
main ---> 2019-04-19 15:49:14.782
test4 ---> 2019-04-19 15:49:19.786
test4 ---> 2019-04-19 15:49:23.293
test4 ---> 2019-04-19 15:49:26.798
test4 ---> 2019-04-19 15:49:30.303
test4 ---> 2019-04-19 15:49:33.812
test4 ---> 2019-04-19 15:49:37.322
test4 ---> 2019-04-19 15:49:40.828
test4 ---> 2019-04-19 15:49:44.333
test4 ---> 2019-04-19 15:49:47.841
test4 ---> 2019-04-19 15:49:51.347
```




