---
layout:     post
title:      关于 BlockingQueue 的一些认识及资料汇总
subtitle:   及个人理解
date:       2019-04-17
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_656.jpg
catalog: true
tags:
    - java集合
---


# 关于`BlockingQueue`的一些认识及资料汇总

## BlockingQueue
```java
public interface BlockingQueue<E> extends Queue<E> 


public abstract class AbstractQueue<E>
    extends AbstractCollection<E>
    implements Queue<E> 
```

阻塞队列 BlockingQueue 支持两个附加操作的 Queue，这两个操作是：`获取元素时等待队列变为非空`，以及`存储元素时等待空间变得可用`。

BlockingQueue 不接受 `null` 元素。试图 add、put 或 offer 一个 null 元素时，某些实现会抛出 NullPointerException。null 被`用作指示 poll 操作失败的警戒值`。

BlockingQueue 可以是限定容量的。默认容量 Integer.MAX_VALUE 。

BlockingQueue 实现是线程安全的。所有排队方法都可以使用内部锁或其他形式的并发控制来自动达到它们的目的。

---

包含的方法：
```
boolean	add(E e) 
          将指定元素插入此队列中（如果立即可行且不会违反容量限制），成功时返回 true，如果当前没有可用的空间，则抛出 IllegalStateException。
boolean	contains(Object o) 
          如果此队列包含指定元素，则返回 true。
int	drainTo(Collection<? super E> c) 
          移除此队列中所有可用的元素，并将它们添加到给定 collection 中。
int	drainTo(Collection<? super E> c, int maxElements) 
          最多从此队列中移除给定数量的可用元素，并将这些元素添加到给定 collection 中。
boolean	offer(E e) 
          将指定元素插入此队列中（如果立即可行且不会违反容量限制），成功时返回 true，如果当前没有可用的空间，则返回 false。
boolean	offer(E e, long timeout, TimeUnit unit) 
          将指定元素插入此队列中，在到达指定的等待时间前等待可用的空间（如果有必要）。
E	poll(long timeout, TimeUnit unit) 
          获取并移除此队列的头部，在指定的等待时间前等待可用的元素（如果有必要）。
void	put(E e) 
          将指定元素插入此队列中，将等待可用的空间（如果有必要）。
int	remainingCapacity() 
          返回在无阻塞的理想情况下（不存在内存或资源约束）此队列能接受的附加元素数量；如果没有内部限制，则返回 Integer.MAX_VALUE。
boolean	remove(Object o) 
          从此队列中移除指定元素的单个实例（如果存在）。
E	take() 
          获取并移除此队列的头部，在元素变得可用之前一直等待（如果有必要）。
```

继承的方法：
```
E poll()
获取并移除此队列的头，如果此队列为空，则返回 null。
返回：
队列的头，如果此队列为空，则返回 null

E peek()
获取但不移除此队列的头；如果此队列为空，则返回 null。
返回：
此队列的头；如果此队列为空，则返回 null
```

## ArrayBlockingQueue

```java
public class ArrayBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable 
```

一个由`数组`支持的`有界阻塞队列`。此队列按 `FIFO（先进先出）原则`对元素进行排序。队列的头部 是在队列中存在时间最长的元素。队列的尾部 是在队列中存在时间最短的元素。新元素插入到队列的尾部，队列获取操作则是从队列头部开始获得元素。

这是一个典型的“`有界缓存区`”，`固定大小`的数组在其中保持生产者插入的元素和使用者提取的元素。一旦创建了这样的缓存区，就不能再增加其容量。试图向已满队列中放入元素会导致操作受阻塞；试图从空队列中提取元素将导致类似阻塞。

此类`支持对等待的生产者线程和使用者线程进行排序的可选公平策略`。默认情况下，不保证是这种排序。然而，通过将公平性 (fairness) 设置为 true 而构造的队列允许按照 FIFO 顺序访问线程。公平性通常会降低吞吐量，但也减少了可变性和避免了“不平衡性”。

---

- 构造方法摘要
```
ArrayBlockingQueue(int capacity) 
          创建一个带有给定的（固定）容量和默认访问策略的 ArrayBlockingQueue。
ArrayBlockingQueue(int capacity, boolean fair) 
          创建一个具有给定的（固定）容量和指定访问策略的 ArrayBlockingQueue。
ArrayBlockingQueue(int capacity, boolean fair, Collection<? extends E> c) 
          创建一个具有给定的（固定）容量和指定访问策略的 ArrayBlockingQueue，它最初包含给定 collection 的元素，并以 collection 迭代器的遍历顺序添加元素。
```

- 方法摘要
```
 boolean	add(E e) 
          将指定的元素插入到此队列的尾部（如果立即可行且不会超过该队列的容量），在成功时返回 true，如果此队列已满，则抛出 IllegalStateException。
 void	clear() 
          自动移除此队列中的所有元素。
 boolean	contains(Object o) 
          如果此队列包含指定的元素，则返回 true。
 int	drainTo(Collection<? super E> c) 
          移除此队列中所有可用的元素，并将它们添加到给定 collection 中。
 int	drainTo(Collection<? super E> c, int maxElements) 
          最多从此队列中移除给定数量的可用元素，并将这些元素添加到给定 collection 中。
 boolean	offer(E e) 
          将指定的元素插入到此队列的尾部（如果立即可行且不会超过该队列的容量），在成功时返回 true，如果此队列已满，则返回 false。
 boolean	offer(E e, long timeout, TimeUnit unit) 
          将指定的元素插入此队列的尾部，如果该队列已满，则在到达指定的等待时间之前等待可用的空间。
 E	peek() 
          获取但不移除此队列的头；如果此队列为空，则返回 null。
 E	poll() 
          获取并移除此队列的头，如果此队列为空，则返回 null。
 E	poll(long timeout, TimeUnit unit) 
          获取并移除此队列的头部，在指定的等待时间前等待可用的元素（如果有必要）。
 void	put(E e) 
          将指定的元素插入此队列的尾部，如果该队列已满，则等待可用的空间。
 int	remainingCapacity() 
          返回在无阻塞的理想情况下（不存在内存或资源约束）此队列能接受的其他元素数量。
 boolean	remove(Object o) 
          从此队列中移除指定元素的单个实例（如果存在）。
 int	size() 
          返回此队列中元素的数量。
 E	take() 
          获取并移除此队列的头部，在元素变得可用之前一直等待（如果有必要）。
 Object[]	toArray() 
          返回一个按适当顺序包含此队列中所有元素的数组。
 <T> T[]     toArray(T[] a) 
          返回一个按适当顺序包含此队列中所有元素的数组；返回数组的运行时类型是指定数组的运行时类型。
```

---

- 案例
```java
import org.junit.Test;
import java.util.concurrent.ArrayBlockingQueue;

public class ArrayBlockingQueueTest {

    /**
     * @see ArrayBlockingQueue 是一种阻塞队列，
     */
    @Test
    public void test1() {
        // 默认非公平队列
        ArrayBlockingQueue queue = new ArrayBlockingQueue(16);

        // 公平队列
        ArrayBlockingQueue fairQueue = new ArrayBlockingQueue(16, true);

    }

    @Test
    public void test2() {
        ArrayBlockingQueue<String> queue = new ArrayBlockingQueue(2);

        // offer 方法 和 add 方法 都可以添加
        queue.offer("111");
        queue.add("222");

        // 使用 offer 方法，若阻塞队列已满，则添加失败，返回 false
        boolean o3 = queue.offer("333");
        System.out.println(o3);

        // 使用 add 方法，若阻塞队列已满，则添加失败，抛出异常 java.lang.IllegalStateException: Queue full
        boolean a = queue.add("333");

    }

    @Test
    public void test3() throws InterruptedException {
        ArrayBlockingQueue<String> queue = new ArrayBlockingQueue(2);

        queue.put("111");
        queue.put("222");

        // 若队列已满，则线程进行阻塞
        queue.put("333");

    }

    @Test
    public void test4() throws InterruptedException {
        ArrayBlockingQueue<String> queue = new ArrayBlockingQueue(10);


        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < 100; i++) {
                try {
                    // put 放入队列，队列已满时线程阻塞
                    queue.put("--->" + i);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        thread1.start();

        Thread thread2 = new Thread(() -> {
            while (true) {
                try {
                    // take 取出节点，队列已满时线程阻塞
                    String node = queue.take();
                    System.out.println(node);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        thread2.start();
        thread2.join();
    }

    @Test
    public void test5() throws InterruptedException {
        ArrayBlockingQueue<String> queue = new ArrayBlockingQueue(10);
        queue.put("111");
        queue.put("222");
        queue.put("333");

        // 取出队列首部的值，并将节点移除队列
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());
    }

    @Test
    public void test6() throws InterruptedException {
        ArrayBlockingQueue<String> queue = new ArrayBlockingQueue(10);
        queue.put("111");
        queue.put("222");
        queue.put("333");

        // 不将节点移除队列
        System.out.println(queue.peek());
        System.out.println(queue.peek());
        System.out.println(queue.peek());
        System.out.println(queue.peek());
    }

}

```


## LinkedBlockingQueue

```java
public class LinkedBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable
```

一个基于已链接节点的、范围任意的 blocking queue。此队列按 FIFO（先进先出）排序元素。队列的头部 是在队列中时间最长的元素。队列的尾部 是在队列中时间最短的元素。新元素插入到队列的尾部，并且队列获取操作会获得位于队列头部的元素。链接队列的吞吐量通常要高于基于数组的队列，但是在大多数并发应用程序中，其可预知的性能要低。

可选的容量范围构造方法参数作为防止队列过度扩展的一种方法。如果未指定容量，则它等于 Integer.MAX_VALUE。除非插入节点会使队列超出容量，否则每次插入后会动态地创建链接节点。

---

- 构造方法摘要
```
LinkedBlockingQueue() 
          创建一个容量为 Integer.MAX_VALUE 的 LinkedBlockingQueue。
LinkedBlockingQueue(Collection<? extends E> c) 
          创建一个容量是 Integer.MAX_VALUE 的 LinkedBlockingQueue，最初包含给定 collection 的元素，元素按该 collection 迭代器的遍历顺序添加。
LinkedBlockingQueue(int capacity) 
          创建一个具有给定（固定）容量的 LinkedBlockingQueue。
```

- 方法摘要
```
 void	clear() 
          从队列彻底移除所有元素。
 int	drainTo(Collection<? super E> c) 
          移除此队列中所有可用的元素，并将它们添加到给定 collection 中。
 int	drainTo(Collection<? super E> c, int maxElements) 
          最多从此队列中移除给定数量的可用元素，并将这些元素添加到给定 collection 中。
 boolean	offer(E e) 
          将指定元素插入到此队列的尾部（如果立即可行且不会超出此队列的容量），在成功时返回 true，如果此队列已满，则返回 false。
 boolean	offer(E e, long timeout, TimeUnit unit) 
          将指定元素插入到此队列的尾部，如有必要，则等待指定的时间以使空间变得可用。
 E	peek() 
          获取但不移除此队列的头；如果此队列为空，则返回 null。
 E	poll() 
          获取并移除此队列的头，如果此队列为空，则返回 null。
 E	poll(long timeout, TimeUnit unit) 
          获取并移除此队列的头部，在指定的等待时间前等待可用的元素（如果有必要）。
 void	put(E e) 
          将指定元素插入到此队列的尾部，如有必要，则等待空间变得可用。
 int	remainingCapacity() 
          返回理想情况下（没有内存和资源约束）此队列可接受并且不会被阻塞的附加元素数量。
 boolean	remove(Object o) 
          从此队列移除指定元素的单个实例（如果存在）。
 int	size() 
          返回队列中的元素个数。
 E	take() 
          获取并移除此队列的头部，在元素变得可用之前一直等待（如果有必要）。
 Object[]	toArray() 
          返回按适当顺序包含此队列中所有元素的数组。
<T> T[] toArray(T[] a) 
          返回按适当顺序包含此队列中所有元素的数组；返回数组的运行时类型是指定数组的运行时类型。
```

---

- 案例
```java
import org.junit.Test;
import java.util.concurrent.LinkedBlockingQueue;

/**
 * @describe: 类描述信息
 * @author: morningcat.zhang
 * @date: 2019/4/16 5:32 PM
 */
public class LinkedBlockingQueueTest {

    /**
     * @see LinkedBlockingQueue 是一种阻塞队列，
     */
    @Test
    public void test1() {

        // 不指定长度
        LinkedBlockingQueue queue = new LinkedBlockingQueue();

        // 指定长度
        LinkedBlockingQueue queueq16 = new LinkedBlockingQueue(16);
    }

    @Test
    public void test2() {
        LinkedBlockingQueue<String> queue = new LinkedBlockingQueue(2);

        // offer 方法 和 add 方法 都可以添加
        queue.offer("111");
        queue.add("222");

        // 使用 offer 方法，若阻塞队列已满，则添加失败，返回 false
        boolean o3 = queue.offer("333");
        System.out.println(o3);

        // 使用 add 方法，若阻塞队列已满，则添加失败，抛出异常 java.lang.IllegalStateException: Queue full
        boolean a = queue.add("333");

    }

    @Test
    public void test3() throws InterruptedException {
        LinkedBlockingQueue<String> queue = new LinkedBlockingQueue(2);

        queue.put("111");
        queue.put("222");

        // 若队列已满，则线程进行阻塞
        queue.put("333");

    }

    @Test
    public void test4() throws InterruptedException {
        LinkedBlockingQueue<String> queue = new LinkedBlockingQueue(10);


        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < 100; i++) {
                try {
                    // put 放入队列，队列已满时线程阻塞
                    queue.put("--->" + i);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        thread1.start();

        Thread thread2 = new Thread(() -> {
            while (true) {
                try {
                    // take 取出节点，队列已满时线程阻塞
                    String node = queue.take();
                    System.out.println(node);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        thread2.start();
        thread2.join();
    }

    @Test
    public void test5() throws InterruptedException {
        LinkedBlockingQueue<String> queue = new LinkedBlockingQueue(10);
        queue.put("111");
        queue.put("222");
        queue.put("333");

        // 取出队列首部的值，并将节点移除队列
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());
    }

    @Test
    public void test6() throws InterruptedException {
        LinkedBlockingQueue<String> queue = new LinkedBlockingQueue(10);
        queue.put("111");
        queue.put("222");
        queue.put("333");

        // 不将节点移除队列
        System.out.println(queue.peek());
        System.out.println(queue.peek());
        System.out.println(queue.peek());
        System.out.println(queue.peek());
    }

    @Test
    public void test7() throws InterruptedException {

    }
}
```

## PriorityBlockingQueue

```java
public class PriorityBlockingQueue<E> extends AbstractQueue<E>
    implements BlockingQueue<E>, java.io.Serializable
```

一个无界阻塞队列，它使用与类 PriorityQueue 相同的顺序规则，并且提供了阻塞获取操作。虽然此队列逻辑上是无界的，但是资源被耗尽时试图执行 add 操作也将失败（导致 OutOfMemoryError）。此类不允许使用 null 元素。依赖自然顺序的优先级队列也不允许插入不可比较的对象（这样做会导致抛出 ClassCastException）。

此类及其迭代器可以实现 Collection 和 Iterator 接口的所有可选 方法。iterator() 方法中提供的迭代器并不 保证以特定的顺序遍历 PriorityBlockingQueue 的元素。如果需要有序地进行遍历，则应考虑使用 Arrays.sort(pq.toArray())。此外，可以使用方法 drainTo 按优先级顺序移除 全部或部分元素，并将它们放在另一个 collection 中。

---

- 构造方法摘要
```
PriorityBlockingQueue() 
          用默认的初始容量 (11) 创建一个 PriorityBlockingQueue，并根据元素的自然顺序对其元素进行排序。
PriorityBlockingQueue(Collection<? extends E> c) 
          创建一个包含指定 collection 元素的 PriorityBlockingQueue。
PriorityBlockingQueue(int initialCapacity) 
          使用指定的初始容量创建一个 PriorityBlockingQueue，并根据元素的自然顺序对其元素进行排序。
PriorityBlockingQueue(int initialCapacity, Comparator<? super E> comparator) 
          使用指定的初始容量创建一个 PriorityBlockingQueue，并根据指定的比较器对其元素进行排序。
```

- 方法摘要
```
 boolean	add(E e) 
          将指定元素插入此优先级队列。
 void	clear() 
          完全移除此队列中的所有元素。
 Comparator<? super E>	comparator() 
          返回用于对此队列元素进行排序的比较器；如果此队列使用其元素的自然顺序，则返回 null。
 boolean	contains(Object o) 
          如果队列包含指定的元素，则返回 true。
 int	drainTo(Collection<? super E> c) 
          移除此队列中所有可用的元素，并将它们添加到给定 collection 中。
 int	drainTo(Collection<? super E> c, int maxElements) 
          最多从此队列中移除给定数量的可用元素，并将这些元素添加到给定 collection 中。
 Iterator<E>	iterator() 
          返回在此队列元素上进行迭代的迭代器。
 boolean	offer(E e) 
          将指定元素插入此优先级队列。
 boolean	offer(E e, long timeout, TimeUnit unit) 
          将指定元素插入此优先级队列。
 E	peek() 
          获取但不移除此队列的头；如果此队列为空，则返回 null。
 E	poll() 
          获取并移除此队列的头，如果此队列为空，则返回 null。
 E	poll(long timeout, TimeUnit unit) 
          获取并移除此队列的头部，在指定的等待时间前等待可用的元素（如果有必要）。
 void	put(E e) 
          将指定元素插入此优先级队列。
 int	remainingCapacity() 
          总是返回 Integer.MAX_VALUE，因为 PriorityBlockingQueue 没有容量限制。
 boolean	remove(Object o) 
          从队列中移除指定元素的单个实例（如果存在）。
 int	size() 
          返回此 collection 中的元素数。
 E	take() 
          获取并移除此队列的头部，在元素变得可用之前一直等待（如果有必要）。
 Object[]	toArray() 
          返回包含此队列所有元素的数组。
<T> T[] toArray(T[] a) 
          返回一个包含此队列所有元素的数组；返回数组的运行时类型是指定数组的类型。
```

- 案例
```java
package collection.queue.block_queue;

import org.junit.Test;

import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.PriorityBlockingQueue;

/**
 * @describe: 类描述信息
 * @author: morningcat.zhang
 * @date: 2019/4/16 7:20 PM
 */
public class PriorityBlockingQueueTest {

    /**
     * @see PriorityBlockingQueue
     */
    @Test
    public void test1() {
        PriorityBlockingQueue queue = new PriorityBlockingQueue();

        PriorityBlockingQueue queue2 = new PriorityBlockingQueue(16);

        /**
         * 根据指定的比较器对其元素进行排序
         */
        PriorityBlockingQueue queue3 = new PriorityBlockingQueue(16, new Comparator() {
            @Override
            public int compare(Object o1, Object o2) {
                return 0;
            }
        });
    }

    private void add(BlockingQueue queue){
        queue.offer("dffg");
        queue.offer("adbg");
        queue.offer("xcff");
    }

    @Test
    public void test2() {
        PriorityBlockingQueue<String> queue = new PriorityBlockingQueue();

        add(queue);

        /**
         * 获取但不移除此队列的头；如果此队列为空，则返回 null。
         */
        System.out.println(queue.peek());
        System.out.println(queue.peek());
        System.out.println(queue.peek());
        System.out.println(queue.peek());
    }

    @Test
    public void test3() {
        PriorityBlockingQueue<String> queue = new PriorityBlockingQueue();

        add(queue);

        /**
         * 获取并移除此队列的头，如果此队列为空，则返回 null。
         */
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());
        System.out.println(queue.poll());
    }

    @Test
    public void test4() throws InterruptedException {
        PriorityBlockingQueue<String> queue = new PriorityBlockingQueue();

        add(queue);

        System.out.println(queue.take());
        System.out.println(queue.take());
        System.out.println(queue.take());
        // 发生阻塞
        System.out.println(queue.take());
    }

    @Test
    public void test5() {
        PriorityBlockingQueue<String> queue = new PriorityBlockingQueue();

        add(queue);

        List<String> list = new ArrayList();
        // 移除此队列中所有可用的元素，并将它们添加到给定 collection 中。
        queue.drainTo(list);
        System.out.println(list);
    }

    @Test
    public void test6() {
        PriorityBlockingQueue<String> queue = new PriorityBlockingQueue();

        add(queue);

        List<String> list = new ArrayList();
        queue.drainTo(list, 3);
        System.out.println(list);
    }

    @Test
    public void test7(){

    }

}


```


## SynchronousQueue

```java
public class SynchronousQueue<E> extends AbstractQueue<E>
    implements BlockingQueue<E>, java.io.Serializable
```

一种阻塞队列，其中每个插入操作必须等待另一个线程的对应移除操作 ，反之亦然。同步队列没有任何内部容量，甚至连一个队列的容量都没有。不能在同步队列上进行 peek，因为仅在试图要移除元素时，该元素才存在；除非另一个线程试图移除某个元素，否则也不能（使用任何方法）插入元素；也不能迭代队列，因为其中没有元素可用于迭代。队列的头 是尝试添加到队列中的首个已排队插入线程的元素；如果没有这样的已排队线程，则没有可用于移除的元素并且 poll() 将会返回 null。对于其他 Collection 方法（例如 contains），SynchronousQueue 作为一个空 collection。此队列不允许 null 元素。

同步队列类似于 CSP 和 Ada 中使用的 rendezvous 信道。它非常适合于`传递性设计`，在这种设计中，在一个线程中运行的对象要将某些信息、事件或任务传递给在另一个线程中运行的对象，它就必须与该对象同步。

对于正在等待的生产者和使用者线程而言，此类支持可选的公平排序策略。默认情况下不保证这种排序。但是，使用公平设置为 true 所构造的队列可保证线程以 FIFO 的顺序进行访问。

--- 

- 构造方法摘要
```
SynchronousQueue() 
          创建一个具有非公平访问策略的 SynchronousQueue。
SynchronousQueue(boolean fair) 
          创建一个具有指定公平策略的 SynchronousQueue。
```

- 方法摘要
```
 void	clear() 
          不执行任何操作。
 boolean	contains(Object o) 
          始终返回 false。
 boolean	containsAll(Collection<?> c) 
          除非给定 collection 为空，否则返回 false。
 int	drainTo(Collection<? super E> c) 
          移除此队列中所有可用的元素，并将它们添加到给定 collection 中。
 int	drainTo(Collection<? super E> c, int maxElements) 
          最多从此队列中移除给定数量的可用元素，并将这些元素添加到给定 collection 中。
 boolean	isEmpty() 
          始终返回 true。
 Iterator<E>	iterator() 
          返回一个空迭代器，其中 hasNext 始终返回 false。
 boolean	offer(E e) 
          如果另一个线程正在等待以便接收指定元素，则将指定元素插入到此队列。
 boolean	offer(E o, long timeout, TimeUnit unit) 
          将指定元素插入到此队列，如有必要则等待指定的时间，以便另一个线程接收它。
 E	peek() 
          始终返回 null。
 E	poll() 
          如果另一个线程当前正要使用某个元素，则获取并移除此队列的头。
 E	poll(long timeout, TimeUnit unit) 
          获取并移除此队列的头，如有必要则等待指定的时间，以便另一个线程插入它。
 void	put(E o) 
          将指定元素添加到此队列，如有必要则等待另一个线程接收它。
 int	remainingCapacity() 
          始终返回 0。
 boolean	remove(Object o) 
          始终返回 false。
 boolean	removeAll(Collection<?> c) 
          始终返回 false。
 boolean	retainAll(Collection<?> c) 
          始终返回 false。
 int	size() 
          始终返回 0。
 E	take() 
          获取并移除此队列的头，如有必要则等待另一个线程插入它。
 Object[]	toArray() 
          返回一个 0 长度的数组。
<T> T[]
toArray(T[] a) 
          将指定数组的第 0 个元素设置为 null（如果该数组有非 0 的长度）并返回它。
```

- 案例

```java
package collection.queue.block_queue;

import org.junit.Test;

import java.util.concurrent.SynchronousQueue;

/**
 * @describe: 类描述信息
 * @author: morningcat.zhang
 * @date: 2019/4/16 7:41 PM
 */
public class SynchronousQueueTest {

    /**
     * @see SynchronousQueue
     */
    @Test
    public void test1() {
        /**
         * 默认非公平锁
         */
        SynchronousQueue queue = new SynchronousQueue();

        SynchronousQueue queue2 = new SynchronousQueue(true);
    }

    @Test
    public void test2() {
        SynchronousQueue queue = new SynchronousQueue();
        queue.offer("xxx");
        System.out.println(queue.size());

        queue.add("xxx");
    }

    @Test
    public void test3() throws InterruptedException {

        SynchronousQueue queue = new SynchronousQueue();

        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < 100; i++) {
                try {
                    queue.put("" + i);

                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        thread1.start();

        Thread thread2 = new Thread(() -> {
            while (true) {

                try {
                    Object o = queue.take();
                    System.out.println(o);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }
        });
        thread2.start();
        thread2.join();

    }
}

```

## DelayQueue

```java
public class DelayQueue<E extends Delayed> extends AbstractQueue<E>
    implements BlockingQueue<E> 
```

Delayed 元素的一个无界阻塞队列，只有在延迟期满时才能从中提取元素。该队列的头部 是延迟期满后保存时间最长的 Delayed 元素。如果延迟都还没有期满，则队列没有头部，并且 poll 将返回 null。当一个元素的 getDelay(TimeUnit.NANOSECONDS) 方法返回一个小于等于 0 的值时，将发生到期。即使无法使用 take 或 poll 移除未到期的元素，也不会将这些元素作为正常元素对待。例如，size 方法同时返回到期和未到期元素的计数。此队列不允许使用 null 元素。

---

- 构造方法摘要
```
DelayQueue() 
          创建一个最初为空的新 DelayQueue。
DelayQueue(Collection<? extends E> c) 
          创建一个最初包含 Delayed 实例的给定 collection 元素的 DelayQueue。
```

- 方法摘要
```
 boolean	add(E e) 
          将指定元素插入此延迟队列中。
 void	clear() 
          自动移除此延迟队列的所有元素。
 int	drainTo(Collection<? super E> c) 
          移除此队列中所有可用的元素，并将它们添加到给定 collection 中。
 int	drainTo(Collection<? super E> c, int maxElements) 
          最多从此队列中移除给定数量的可用元素，并将这些元素添加到给定 collection 中。
 Iterator<E>	iterator() 
          返回在此队列所有元素（既包括到期的，也包括未到期的）上进行迭代的迭代器。
 boolean	offer(E e) 
          将指定元素插入此延迟队列。
 boolean	offer(E e, long timeout, TimeUnit unit) 
          将指定元素插入此延迟队列中。
 E	peek() 
          获取但不移除此队列的头部；如果此队列为空，则返回 null。
 E	poll() 
          获取并移除此队列的头，如果此队列不包含具有已到期延迟时间的元素，则返回 null。
 E	poll(long timeout, TimeUnit unit) 
          获取并移除此队列的头部，在可从此队列获得到期延迟的元素，或者到达指定的等待时间之前一直等待（如有必要）。
 void	put(E e) 
          将指定元素插入此延迟队列。
 int	remainingCapacity() 
          因为 DelayQueue 没有容量限制，所以它总是返回 Integer.MAX_VALUE。
 boolean	remove(Object o) 
          从此队列中移除指定元素的单个实例（如果存在），无论它是否到期。
 int	size() 
          返回此 collection 中的元素数。
 E	take() 
          获取并移除此队列的头部，在可从此队列获得到期延迟的元素之前一直等待（如有必要）。
 Object[]	toArray() 
          返回包含此队列所有元素的数组。
<T> T[]
toArray(T[] a) 
          返回一个包含此队列所有元素的数组；返回数组的运行时类型是指定数组的类型。
```



