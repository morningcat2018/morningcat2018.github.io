---
layout:     post
title:      关于 ThreadPoolExecutor 的一些资料汇总
subtitle:   及个人理解
date:       2019-04-19
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_656.jpg
catalog: true
tags:
    - java
---

关于`ThreadPoolExecutor`的一些资料汇总及个人理解

相关资料：
- [关于 BlockingQueue 的一些认识及资料汇总](https://blog.csdn.net/u013837825/article/details/89362121)
- [线程池ThreadPoolExecutor的拒绝策略](https://blog.csdn.net/u013837825/article/details/89385446)

## 1.资料汇总

```java

/**
 * @since 1.5
 * @author Doug Lea
 */
public class ThreadPoolExecutor extends AbstractExecutorService {
    // ...
}
```

- 构造方法

```
ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) 
用给定的初始参数和默认的线程工厂及被拒绝的执行处理程序创建新的 ThreadPoolExecutor。 

ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, RejectedExecutionHandler handler) 
用给定的初始参数和默认的线程工厂创建新的 ThreadPoolExecutor。 

ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory) 
用给定的初始参数和默认被拒绝的执行处理程序创建新的 ThreadPoolExecutor。 

ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler) 
用给定的初始参数创建新的 ThreadPoolExecutor。 
```

- 参数说明

```
corePoolSize - 池中所保存的线程数，包括空闲线程。
maximumPoolSize - 池中允许的最大线程数。
keepAliveTime - 当线程数大于核心时，此为终止空闲线程等待新任务的最长时间。
unit - keepAliveTime 参数的时间单位。
workQueue - 执行前用于保持任务的队列。此队列仅保持由 execute 方法提交的 Runnable 任务。 
threadFactory - 执行程序创建新线程时使用的工厂。
handler - (拒绝策略)由于超出线程范围和队列容量而使执行被阻塞时所使用的处理程序。 
```

- 抛出异常

```
IllegalArgumentException - 如果 corePoolSize 或 keepAliveTime 小于 0，或者 maximumPoolSize 小于等于 0，或者 corePoolSize 大于 maximumPoolSize。 
NullPointerException - 如果 workQueue、threadFactory 或 handler 为 null。
```

- 核心概念

```
1.核心和最大池大小 
ThreadPoolExecutor 将根据 corePoolSize 和 maximumPoolSize 设置的边界自动调整池大小。当新任务在方法 execute(java.lang.Runnable) 中提交时，如果运行的线程少于 corePoolSize，则创建新线程来处理请求，即使其他辅助线程是空闲的。如果运行的线程多于 corePoolSize 而少于 maximumPoolSize，则仅当队列满时才创建新线程。如果设置的 corePoolSize 和 maximumPoolSize 相同，则创建了固定大小的线程池。如果将 maximumPoolSize 设置为基本的无界值（如 Integer.MAX_VALUE），则允许池适应任意数量的并发任务。在大多数情况下，核心和最大池大小仅基于构造来设置，不过也可以使用 setCorePoolSize(int) 和 setMaximumPoolSize(int) 进行动态更改。

2.按需构造 
默认情况下，即使核心线程最初只是在新任务到达时才创建和启动的，也可以使用方法 prestartCoreThread() 或 prestartAllCoreThreads() 对其进行动态重写。如果构造带有非空队列的池，则可能希望预先启动线程。 

3.创建新线程 
使用 ThreadFactory 创建新线程。如果没有另外说明，则在同一个 ThreadGroup 中一律使用 Executors.defaultThreadFactory() 创建线程，并且这些线程具有相同的 NORM_PRIORITY 优先级和非守护进程状态。通过提供不同的 ThreadFactory，可以改变线程的名称、线程组、优先级、守护进程状态，等等。如果从 newThread 返回 null 时 ThreadFactory 未能创建线程，则执行程序将继续运行，但不能执行任何任务。 

4.保持活动时间 
如果池中当前有多于 corePoolSize 的线程，则这些多出的线程在空闲时间超过 keepAliveTime 时将会终止（参见 getKeepAliveTime(java.util.concurrent.TimeUnit)）。这提供了当池处于非活动状态时减少资源消耗的方法。如果池后来变得更为活动，则可以创建新的线程。也可以使用方法 setKeepAliveTime(long, java.util.concurrent.TimeUnit) 动态地更改此参数。使用 Long.MAX_VALUE TimeUnit.NANOSECONDS 的值在关闭前有效地从以前的终止状态禁用空闲线程。默认情况下，保持活动策略只在有多于 corePoolSizeThreads 的线程时应用。但是只要 keepAliveTime 值非 0，allowCoreThreadTimeOut(boolean) 方法也可将此超时策略应用于核心线程。 

5.排队 
所有 BlockingQueue 都可用于传输和保持提交的任务。可以使用此队列与池大小进行交互： 
如果运行的线程少于 corePoolSize，则 Executor 始终首选添加新的线程，而不进行排队。 
如果运行的线程等于或多于 corePoolSize，则 Executor 始终首选将请求加入队列，而不添加新的线程。 
如果无法将请求加入队列，则创建新的线程，除非创建此线程超出 maximumPoolSize，在这种情况下，任务将被拒绝。 
排队有三种通用策略： 
直接提交。工作队列的默认选项是 SynchronousQueue，它将任务直接提交给线程而不保持它们。在此，如果不存在可用于立即运行任务的线程，则试图把任务加入队列将失败，因此会构造一个新的线程。此策略可以避免在处理可能具有内部依赖性的请求集时出现锁。直接提交通常要求无界 maximumPoolSizes 以避免拒绝新提交的任务。当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。 
无界队列。使用无界队列（例如，不具有预定义容量的 LinkedBlockingQueue）将导致在所有 corePoolSize 线程都忙时新任务在队列中等待。这样，创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize 的值也就无效了。）当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web 页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。 
有界队列。当使用有限的 maximumPoolSizes 时，有界队列（如 ArrayBlockingQueue）有助于防止资源耗尽，但是可能较难调整和控制。队列大小和最大池大小可能需要相互折衷：使用大型队列和小型池可以最大限度地降低 CPU 使用率、操作系统资源和上下文切换开销，但是可能导致人工降低吞吐量。如果任务频繁阻塞（例如，如果它们是 I/O 边界），则系统可能为超过您许可的更多线程安排时间。使用小型队列通常要求较大的池大小，CPU 使用率较高，但是可能遇到不可接受的调度开销，这样也会降低吞吐量。 

6.被拒绝的任务 
当 Executor 已经关闭，并且 Executor 将有限边界用于最大线程和工作队列容量，且已经饱和时，在方法 execute(java.lang.Runnable) 中提交的新任务将被拒绝。在以上两种情况下，execute 方法都将调用其 RejectedExecutionHandler 的 RejectedExecutionHandler.rejectedExecution(java.lang.Runnable, java.util.concurrent.ThreadPoolExecutor) 方法。下面提供了四种预定义的处理程序策略： 
在默认的 ThreadPoolExecutor.AbortPolicy 中，处理程序遭到拒绝将抛出运行时 RejectedExecutionException。 
在 ThreadPoolExecutor.CallerRunsPolicy 中，线程调用运行该任务的 execute 本身。此策略提供简单的反馈控制机制，能够减缓新任务的提交速度。 
在 ThreadPoolExecutor.DiscardPolicy 中，不能执行的任务将被删除。 
在 ThreadPoolExecutor.DiscardOldestPolicy 中，如果执行程序尚未关闭，则位于工作队列头部的任务将被删除，然后重试执行程序（如果再次失败，则重复此过程）。 
定义和使用其他种类的 RejectedExecutionHandler 类也是可能的，但这样做需要非常小心，尤其是当策略仅用于特定容量或排队策略时。 

7.钩子 (hook) 方法 
此类提供 protected 可重写的 beforeExecute(java.lang.Thread, java.lang.Runnable) 和 afterExecute(java.lang.Runnable, java.lang.Throwable) 方法，这两种方法分别在执行每个任务之前和之后调用。它们可用于操纵执行环境；例如，重新初始化 ThreadLocal、搜集统计信息或添加日志条目。此外，还可以重写方法 terminated() 来执行 Executor 完全终止后需要完成的所有特殊处理。 
如果钩子 (hook) 或回调方法抛出异常，则内部辅助线程将依次失败并突然终止。 

8.队列维护 
方法 getQueue() 允许出于监控和调试目的而访问工作队列。强烈反对出于其他任何目的而使用此方法。remove(java.lang.Runnable) 和 purge() 这两种方法可用于在取消大量已排队任务时帮助进行存储回收。 

9.终止 
程序 AND 不再引用的池没有剩余线程会自动 shutdown。如果希望确保回收取消引用的池（即使用户忘记调用 shutdown()），则必须安排未使用的线程最终终止：设置适当保持活动时间，使用 0 核心线程的下边界和/或设置 allowCoreThreadTimeOut(boolean)。 
```

## 2.个人理解

以下内容纯属个人理解，若有误，请指教：

#### 1. 最初

刚创建线程池时，线程池中的`线程资源数`为 0 ；

#### 2. 提交任务

执行 execute(Runnable command) 提交任务时，当 线程池中的`线程资源数` 小于 `核心线程数(corePoolSize)` 时，创建新线程并放入线程池。
若指定了 线程工厂类（ThreadFactory），则使用指定的 ThreadFactory 创建新线程；若没有指定，使用默认的线程工厂（Executors.defaultThreadFactory()）创建新线程。
注意：在上述情况下（getPoolSize() < getCorePoolSize()），即使核心线程空闲，当新任务到达时也会创建新线程。
```java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }

    // Executors.defaultThreadFactory()
    public static ThreadFactory defaultThreadFactory() {
        return new DefaultThreadFactory();
    }

    /**
     * The default thread factory (Executors 的内部类)
     */
    static class DefaultThreadFactory implements ThreadFactory {
        private static final AtomicInteger poolNumber = new AtomicInteger(1);
        private final ThreadGroup group;
        private final AtomicInteger threadNumber = new AtomicInteger(1);
        private final String namePrefix;

        DefaultThreadFactory() {
            SecurityManager s = System.getSecurityManager();
            group = (s != null) ? s.getThreadGroup() :
                                  Thread.currentThread().getThreadGroup();
            namePrefix = "pool-" +
                          poolNumber.getAndIncrement() +
                         "-thread-";
        }

        public Thread newThread(Runnable r) {
            Thread t = new Thread(group, r,
                                  namePrefix + threadNumber.getAndIncrement(),
                                  0);
            if (t.isDaemon())
                t.setDaemon(false);
            if (t.getPriority() != Thread.NORM_PRIORITY)
                t.setPriority(Thread.NORM_PRIORITY);
            return t;
        }
    }
```

执行 execute(Runnable command) 提交任务时，当 线程池中的`线程资源数` 大于等于 `核心线程数(corePoolSize)` 时，且`核心线程`并未处在空闲时期；
若任务队列（workQueue）尚有空间，则将新任务放入任务队列中等待执行；
若任务队列为`无限队列`（Integer.MAX_VALUE），则会一直放入任务队列中；
若任务队列为 PriorityBlockingQueue ，队列空间大小本身就是 0 ，即队列不会存放任务。
```java
LinkedBlockingQueue queue = new LinkedBlockingQueue();

    public LinkedBlockingQueue() {
        this(Integer.MAX_VALUE);
    }
```

#### 3. 任务队列已无剩余空间

执行 execute(Runnable command) 提交任务时，当`核心线程`全部未处在空闲时期，且`任务队列`已无剩余空间；

若 指定`最大线程数`（maximumPoolSize） > 核心线程数（corePoolSize）：
则会通过`线程工厂`继续创建线程并放入线程池；

#### 4. 拒绝任务

执行 execute(Runnable command) 提交任务时：
`核心线程`全部未处在空闲时期；
`任务队列`已无剩余空间；
线程池已达到`最大线程数`；
则线程池会拒绝掉提交的任务...

若指定了`拒绝策略`，则按照指定策略的执行；参考[线程池ThreadPoolExecutor的拒绝策略](https://blog.csdn.net/u013837825/article/details/89385446)
若未指定，默认拒绝策略为 `AbortPolicy`, 抛出异常（个人不推荐使用）
```java
private static final RejectedExecutionHandler defaultHandler = new AbortPolicy();


public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    throw new RejectedExecutionException("Task " + r.toString() +
                                            " rejected from " +
                                            e.toString());
}
```

#### 5. 当线程池池空闲下来后

因临时需要扩展出来的线程（最大线程数 - 核心线程数）会被释放。

最佳实践就是指定 `核心线程数` 等于 `最大线程数`，则不会因为重复的创建、销毁线程浪费系统资源。

## 3.个人实践

```java
package collection.queue.thread_pool;

import org.junit.Test;

import java.io.IOException;
import java.util.Random;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * @describe: 类描述信息
 * @author: morningcat.zhang
 * @date: 2019/4/18 1:02 PM
 */
public class ExecutorTest {

    @Test
    public void test1() throws IOException {
        ArrayBlockingQueue workQueue = new ArrayBlockingQueue(20);
        ThreadPoolExecutor executorService = new ThreadPoolExecutor(5, 20, 1, TimeUnit.SECONDS, workQueue, new TestThreadFactory(), new TestRejectedExecutionHandler());
        for (int i = 0; i < 50; i++) {
            executorService.execute(() -> {
                System.out.println(Thread.currentThread().getName() + " ---> 添加任务");
                try {
                    Thread.sleep(new Random().nextInt(5000));
                    //Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });

            System.out.println(Thread.currentThread().getName() + "------ " + i + " ---> ");
        }

        new Thread(() -> {
            // executorService.getActiveCount() > 0
            while (true) {
                System.out.println(Thread.currentThread().getName() + " ------> " + executorService.getPoolSize() + " ---> " + executorService.getActiveCount());
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "【监控线程】").start();

        System.in.read();

    }

    @Test
    public void test2() {

    }

    @Test
    public void test3() {

    }
}


package collection.queue.thread_pool;

import java.util.concurrent.BlockingQueue;
import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.ThreadPoolExecutor;

/**
 * @describe: 类描述信息
 * @author: morningcat.zhang
 * @date: 2019/4/18 4:44 PM
 */
public class TestRejectedExecutionHandler implements RejectedExecutionHandler {
    @Override
    public void rejectedExecution(Runnable runnable, ThreadPoolExecutor executor) {
        System.out.println(Thread.currentThread().getName() + "----------------------------> 进入拒绝策略");

        // 延迟执行
        try {
            Thread.sleep(5L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // 放入队列
        BlockingQueue<Runnable> workQueue = executor.getQueue();
        if (workQueue.offer(runnable)) {
            return;
        }

        // 新建线程执行
        new Thread(runnable, "【拒绝策略线程】").start();

        // OR
        // 放弃此任务，记录日志


    }
}



package collection.queue.thread_pool;

import java.util.concurrent.ThreadFactory;

/**
 * @describe: 类描述信息
 * @author: morningcat.zhang
 * @date: 2019/4/18 5:39 PM
 */
public class TestThreadFactory implements ThreadFactory {
    @Override
    public Thread newThread(Runnable runnable) {
        Thread thread = new Thread(runnable, "【线程工厂生产线程】");
        // thread.setPriority(2);
        // thread.setDaemon(true);
        return thread;
    }
}


```

执行结果
```
main------ 0 ---> 
【线程工厂生产线程】 ---> 添加任务
main------ 1 ---> 
【线程工厂生产线程】 ---> 添加任务
main------ 2 ---> 
【线程工厂生产线程】 ---> 添加任务
main------ 3 ---> 
【线程工厂生产线程】 ---> 添加任务
main------ 4 ---> 
【线程工厂生产线程】 ---> 添加任务
main------ 5 ---> 
main------ 6 ---> 
main------ 7 ---> 
main------ 8 ---> 
main------ 9 ---> 
main------ 10 ---> 
main------ 11 ---> 
main------ 12 ---> 
main------ 13 ---> 
main------ 14 ---> 
main------ 15 ---> 
main------ 16 ---> 
main------ 17 ---> 
main------ 18 ---> 
main------ 19 ---> 
main------ 20 ---> 
main------ 21 ---> 
main------ 22 ---> 
main------ 23 ---> 
main------ 24 ---> 
main------ 25 ---> 
【线程工厂生产线程】 ---> 添加任务
main------ 26 ---> 
【线程工厂生产线程】 ---> 添加任务
main------ 27 ---> 
【线程工厂生产线程】 ---> 添加任务
main------ 28 ---> 
【线程工厂生产线程】 ---> 添加任务
【线程工厂生产线程】 ---> 添加任务
main------ 29 ---> 
main------ 30 ---> 
【线程工厂生产线程】 ---> 添加任务
main------ 31 ---> 
【线程工厂生产线程】 ---> 添加任务
main------ 32 ---> 
【线程工厂生产线程】 ---> 添加任务
main------ 33 ---> 
【线程工厂生产线程】 ---> 添加任务
main------ 34 ---> 
【线程工厂生产线程】 ---> 添加任务
main------ 35 ---> 
【线程工厂生产线程】 ---> 添加任务
【线程工厂生产线程】 ---> 添加任务
main------ 36 ---> 
main------ 37 ---> 
【线程工厂生产线程】 ---> 添加任务
main------ 38 ---> 
【线程工厂生产线程】 ---> 添加任务
main------ 39 ---> 
【线程工厂生产线程】 ---> 添加任务
main----------------------------> 进入拒绝策略
main------ 40 ---> 
【拒绝策略线程】 ---> 添加任务
main----------------------------> 进入拒绝策略
main------ 41 ---> 
main----------------------------> 进入拒绝策略
【拒绝策略线程】 ---> 添加任务
【拒绝策略线程】 ---> 添加任务
main------ 42 ---> 
main----------------------------> 进入拒绝策略
main------ 43 ---> 
main----------------------------> 进入拒绝策略
【拒绝策略线程】 ---> 添加任务
main------ 44 ---> 
main----------------------------> 进入拒绝策略
【拒绝策略线程】 ---> 添加任务
main------ 45 ---> 
main----------------------------> 进入拒绝策略
【拒绝策略线程】 ---> 添加任务
main------ 46 ---> 
【拒绝策略线程】 ---> 添加任务
main----------------------------> 进入拒绝策略
main------ 47 ---> 
main----------------------------> 进入拒绝策略
【拒绝策略线程】 ---> 添加任务
main------ 48 ---> 
【拒绝策略线程】 ---> 添加任务
main----------------------------> 进入拒绝策略
main------ 49 ---> 
【拒绝策略线程】 ---> 添加任务
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【线程工厂生产线程】 ---> 添加任务
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【线程工厂生产线程】 ---> 添加任务
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【线程工厂生产线程】 ---> 添加任务
【线程工厂生产线程】 ---> 添加任务
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【线程工厂生产线程】 ---> 添加任务
【监控线程】 ------> 20 ---> 20
【线程工厂生产线程】 ---> 添加任务
【监控线程】 ------> 20 ---> 20
【线程工厂生产线程】 ---> 添加任务
【监控线程】 ------> 20 ---> 20
【线程工厂生产线程】 ---> 添加任务
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【线程工厂生产线程】 ---> 添加任务
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【线程工厂生产线程】 ---> 添加任务
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【线程工厂生产线程】 ---> 添加任务
【线程工厂生产线程】 ---> 添加任务
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【线程工厂生产线程】 ---> 添加任务
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【线程工厂生产线程】 ---> 添加任务
【监控线程】 ------> 20 ---> 20
【线程工厂生产线程】 ---> 添加任务
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 20
【线程工厂生产线程】 ---> 添加任务
【监控线程】 ------> 20 ---> 20
【线程工厂生产线程】 ---> 添加任务
【线程工厂生产线程】 ---> 添加任务
【监控线程】 ------> 20 ---> 20
【线程工厂生产线程】 ---> 添加任务
【线程工厂生产线程】 ---> 添加任务
【监控线程】 ------> 20 ---> 20
【监控线程】 ------> 20 ---> 19
【监控线程】 ------> 20 ---> 18
【监控线程】 ------> 20 ---> 17
【监控线程】 ------> 20 ---> 17
【监控线程】 ------> 20 ---> 17
【监控线程】 ------> 20 ---> 17
【监控线程】 ------> 20 ---> 16
【监控线程】 ------> 20 ---> 13
【监控线程】 ------> 20 ---> 12
【监控线程】 ------> 20 ---> 9
【监控线程】 ------> 19 ---> 7
【监控线程】 ------> 18 ---> 7
【监控线程】 ------> 17 ---> 6
【监控线程】 ------> 17 ---> 6
【监控线程】 ------> 17 ---> 5
【监控线程】 ------> 17 ---> 5
【监控线程】 ------> 14 ---> 5
【监控线程】 ------> 12 ---> 5
【监控线程】 ------> 11 ---> 4
【监控线程】 ------> 8 ---> 3
【监控线程】 ------> 7 ---> 3
【监控线程】 ------> 7 ---> 3
【监控线程】 ------> 6 ---> 3
【监控线程】 ------> 6 ---> 2
【监控线程】 ------> 5 ---> 2
【监控线程】 ------> 5 ---> 2
【监控线程】 ------> 5 ---> 2
【监控线程】 ------> 5 ---> 2
【监控线程】 ------> 5 ---> 1
【监控线程】 ------> 5 ---> 1
【监控线程】 ------> 5 ---> 1
【监控线程】 ------> 5 ---> 1
【监控线程】 ------> 5 ---> 1
【监控线程】 ------> 5 ---> 1
【监控线程】 ------> 5 ---> 1
【监控线程】 ------> 5 ---> 1
【监控线程】 ------> 5 ---> 1
【监控线程】 ------> 5 ---> 1
【监控线程】 ------> 5 ---> 1
【监控线程】 ------> 5 ---> 1
【监控线程】 ------> 5 ---> 1
【监控线程】 ------> 5 ---> 1
【监控线程】 ------> 5 ---> 0
【监控线程】 ------> 5 ---> 0
【监控线程】 ------> 5 ---> 0
【监控线程】 ------> 5 ---> 0
【监控线程】 ------> 5 ---> 0
【监控线程】 ------> 5 ---> 0
【监控线程】 ------> 5 ---> 0
【监控线程】 ------> 5 ---> 0
【监控线程】 ------> 5 ---> 0
【监控线程】 ------> 5 ---> 0
【监控线程】 ------> 5 ---> 0
【监控线程】 ------> 5 ---> 0

Process finished with exit code 130 (interrupted by signal 2: SIGINT)

```

---





