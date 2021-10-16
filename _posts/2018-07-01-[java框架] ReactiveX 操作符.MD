---
layout:     post
title:      ReactiveX 操作符
subtitle:   
date:       2019-10-12
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - java框架
---

ReactiveX的每种编程语言的实现都实现了一组操作符的集合。不同的实现之间有很多重叠的部分，也有一些操作符只存在特定的实现中。每种实现都倾向于用那种编程语言中他们熟悉的上下文中相似的方法给这些操作符命名。

## 创建操作

创建Observable的各种方法。

- just( ) — 将一个或多个对象转换成发射这个或这些对象的一个Observable
- from( ) — 将一个Iterable, 一个Future, 或者一个数组转换成一个Observable
- repeat( ) — 创建一个重复发射指定数据或数据序列的Observable
- repeatWhen( ) — 创建一个重复发射指定数据或数据序列的Observable，它依赖于另一个Observable发射的数据
- create( ) — 使用一个函数从头创建一个Observable
- defer( ) — 只有当订阅者订阅才创建Observable；为每个订阅创建一个新的Observable
- range( ) — 创建一个发射指定范围的整数序列的Observable
- interval( ) — 创建一个按照给定的时间间隔发射整数序列的Observable
- timer( ) — 创建一个在给定的延时之后发射单个数据的Observable
- empty( ) — 创建一个什么都不做直接通知完成的Observable
- error( ) — 创建一个什么都不做直接通知错误的Observable
- never( ) — 创建一个不发射任何数据的Observable

### from

```java
        String[] array = {"a", "b", "c"};
        Observable.fromArray(array)
                .subscribe(System.out::println);

        List<Integer> list = Arrays.asList(1, 2, 3, 45);
        Observable.fromIterable(list)
                .subscribe(System.out::println);
```

```java
        ExecutorService executor = Executors.newFixedThreadPool(3);
        Future<String> future = executor.submit(
                new Callable<String>() {
                    @Override
                    public String call() throws Exception {
                        return "hello world";
                    }
                }
        );
        Observable.fromFuture(future).subscribe(System.out::println);
```

```java
        Observable.fromPublisher(new Publisher<String>() {
            @Override
            public void subscribe(Subscriber<? super String> subscriber) {
                subscriber.onNext("fromPublisher");
            }
        }).subscribe(System.out::println);
```

### just

```java
        Observable.just("a", "b", "c")
                .map(s -> "hello " + s)
                .subscribe(System.out::println);
```

### create

```java
        Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> observableEmitter) throws Throwable {
                try {
                    observableEmitter.onNext("world");
                    observableEmitter.onComplete();
                    //throw new RuntimeException("this is a question.");
                } catch (Throwable t) {
                    observableEmitter.onNext("error:" + t.toString());
                }

            }
        }).subscribe(System.out::println);

        // world
```

### defer

```java
        Observable.defer(new Supplier<ObservableSource<String>>() {

            @Override
            public ObservableSource<String> get() throws Throwable {
                return new ObservableSource<String>() {
                    @Override
                    public void subscribe(Observer<? super String> observer) {
                        observer.onNext("world");
                        observer.onComplete();
                    }
                };
            }
        }).subscribe(System.out::println);

        // world
```

### interval

```java
        Observable.interval(1, TimeUnit.SECONDS).subscribe(System.out::println);

        TimeUnit.SECONDS.sleep(10);

        // 0 1 2 3 4 5 6 7 8 9
```

### range

```java
        Observable.range(10, 10).subscribe(System.out::println);

        TimeUnit.SECONDS.sleep(3);

        // 10 11 12 13 ... 19
```

### timer

```java
        Observable.timer(1, TimeUnit.SECONDS).subscribe(System.out::println);

        TimeUnit.SECONDS.sleep(10);

        // 0
```

### empty never error

```java
        // 创建一个不发射任何数据但是正常终止的Observable
        Observable o1 = Observable.empty();

        // 创建一个不发射数据也不终止的Observable
        Observable o2 = Observable.never();

        // 创建一个不发射数据以一个错误终止的Observable
        Observable o3 = Observable.error(new RuntimeException("rxjava error."));
```

## 变换操作

对Observable发射的数据执行变换操作的各种操作符。

- map( ) — 对序列的每一项都应用一个函数来变换Observable发射的数据序列
- flatMap( ), concatMap( ), and flatMapIterable( ) — 将Observable发射的数据集合变换为Observables集合，然后将这些Observable发射的数据平坦化的放进一个单独的Observable
- switchMap( ) — 将Observable发射的数据集合变换为Observables集合，然后只发射这些Observables最近发射的数据
- scan( ) — 对Observable发射的每一项数据应用一个函数，然后按顺序依次发射每一个值
- groupBy( ) — 将Observable分拆为Observable集合，将原始Observable发射的数据按Key分组，每一个Observable发射一组不同的数据
- buffer( ) — 它定期从Observable收集数据到一个集合，然后把这些数据集合打包发射，而不是一次发射一个
- window( ) — 定期将来自Observable的数据分拆成一些Observable窗口，然后发射这些窗口，而不是每次发射一项
- cast( ) — 在发射之前强制将Observable发射的所有数据转换为指定类型

### map

```java
        Observable.just("a", "b", "c")
                .map(s -> "hello " + s)
                .subscribe(System.out::println);
```

### scan

```java
        // Scan操作符对原始Observable发射的第一项数据应用一个函数，然后将那个函数的结果作为自己的第一项数据发射。它将函数的结果同第二项数据一起填充给这个函数来产生它自己的第二项数据。它持续进行这个过程来产生剩余的数据序列。这个操作符在某些情况下被叫做accumulator
        Observable.range(1, 100)
                .scan((s, v) -> s + v)
                .subscribe(System.out::println);
```

### buffer

```java
        // 定期收集Observable的数据放进一个数据包裹，然后发射这些数据包裹，而不是一次发射一个值。
        // buffer(count)以列表(List)的形式发射非重叠的缓存，每一个缓存至多包含来自原始Observable的count项数据（最后发射的列表数据可能少于count项）
        Observable.just("a", "b", "c", "g", "q")
                .buffer(2)
                .subscribe(System.out::println);
```

```java
       // buffer(count, skip)从原始Observable的第一项数据开始创建新的缓存，此后每当收到skip项数据，用count项数据填充缓存：开头的一项和后续的count-1项，它以列表(List)的形式发射缓存，取决于count和skip的值，这些缓存可能会有重叠部分（比如skip < count时），也可能会有间隙（比如skip > count时）。
        Observable.just("a", "b", "c", "g", "q")
                .buffer(2,3)
                .subscribe(System.out::println);
```

### window

```java
       // 定期将来自原始Observable的数据分解为一个Observable窗口，发射这些窗口，而不是每次发射一项数据
        // Window和Buffer类似，但不是发射来自原始Observable的数据包，它发射的是Observables
        Observable.just("a", "b", "c", "g", "q")
                .window(2)
                .subscribe(System.out::println);
```

### groupBy

```java
        Observable.just("a", "b", "c", "g", "q")
                .groupBy(new Function<String, Integer>() {
                    @Override
                    public Integer apply(String s) throws Throwable {
                        return Integer.valueOf(s.toCharArray()[0]).intValue();
                    }
                })
                .subscribe(new Consumer<GroupedObservable<Integer, String>>() {
                    @Override
                    public void accept(GroupedObservable<Integer, String> integerStringGroupedObservable) throws Throwable {
                        System.out.println(integerStringGroupedObservable.getKey());
                    }
                });
```

### flatMap

```java
   Observable.just("a", "b", "c", "g", "q")
                .flatMap(v -> Observable.just(v).map(w -> "hello " + w))
                .subscribe(System.out::println);
```

## 过滤操作

可用于过滤和选择Observable发射的数据序列

- filter( ) — 过滤数据
- takeLast( ) — 只发射最后的N项数据
- last( ) — 只发射最后的一项数据
- lastOrDefault( ) — 只发射最后的一项数据，如果Observable为空就发射默认值
- takeLastBuffer( ) — 将最后的N项数据当做单个数据发射
- skip( ) — 跳过开始的N项数据
- skipLast( ) — 跳过最后的N项数据
- take( ) — 只发射开始的N项数据
- first( ) and takeFirst( ) — 只发射第一项数据，或者满足某种条件的第一项数据
- firstOrDefault( ) — 只发射第一项数据，如果Observable为空就发射默认值
- elementAt( ) — 发射第N项数据
- elementAtOrDefault( ) — 发射第N项数据，如果Observable数据少于N项就发射默认值
- sample( ) or throttleLast( ) — 定期发射Observable最近的数据
- throttleFirst( ) — 定期发射Observable发射的第一项数据
- throttleWithTimeout( ) or debounce( ) — 只有当Observable在指定的时间后还没有发射数据时，才发射一个数据
- timeout( ) — 如果在一个指定的时间段后还没发射数据，就发射一个异常
- distinct( ) — 过滤掉重复数据
- distinctUntilChanged( ) — 过滤掉连续重复的数据
- ofType( ) — 只发射指定类型的数据
- ignoreElements( ) — 丢弃所有的正常数据，只发射错误或完成通知

## 结合操作

可用于组合多个Observables。

startWith( ) — 在数据序列的开头增加一项数据
merge( ) — 将多个Observable合并为一个
mergeDelayError( ) — 合并多个Observables，让没有错误的Observable都完成后再发射错误通知
zip( ) — 使用一个函数组合多个Observable发射的数据集合，然后再发射这个结果
and( ), then( ), and when( ) — (rxjava-joins) 通过模式和计划组合多个Observables发射的数据集合
combineLatest( ) — 当两个Observables中的任何一个发射了一个数据时，通过一个指定的函数组合每个Observable发射的最新数据（一共两个数据），然后发射这个函数的结果
join( ) and groupJoin( ) — 无论何时，如果一个Observable发射了一个数据项，只要在另一个Observable发射的数据项定义的时间窗口内，就将两个Observable发射的数据合并发射
switchOnNext( ) — 将一个发射Observables的Observable转换成另一个Observable，后者发射这些Observables最近发射的数据

### join


### merge


### and

### startWith


### combineLatest



## 辅助操作

用于Observable的辅助操作符

materialize( ) — 将Observable转换成一个通知列表convert an Observable into a list of Notifications
dematerialize( ) — 将上面的结果逆转回一个Observable
timestamp( ) — 给Observable发射的每个数据项添加一个时间戳
serialize( ) — 强制Observable按次序发射数据并且要求功能是完好的
cache( ) — 记住Observable发射的数据序列并发射相同的数据序列给后续的订阅者
observeOn( ) — 指定观察者观察Observable的调度器
subscribeOn( ) — 指定Observable执行任务的调度器
doOnEach( ) — 注册一个动作，对Observable发射的每个数据项使用
doOnCompleted( ) — 注册一个动作，对正常完成的Observable使用
doOnError( ) — 注册一个动作，对发生错误的Observable使用
doOnTerminate( ) — 注册一个动作，对完成的Observable使用，无论是否发生错误
doOnSubscribe( ) — 注册一个动作，在观察者订阅时使用
doOnUnsubscribe( ) — 注册一个动作，在观察者取消订阅时使用
finallyDo( ) — 注册一个动作，在Observable完成时使用
delay( ) — 延时发射Observable的结果
delaySubscription( ) — 延时处理订阅请求
timeInterval( ) — 定期发射数据
using( ) — 创建一个只在Observable生命周期存在的资源
single( ) — 强制返回单个数据，否则抛出异常
singleOrDefault( ) — 如果Observable完成时返回了单个数据，就返回它，否则返回默认数据
toFuture( ), toIterable( ), toList( ) — 将Observable转换为其它对象或数据结构

## 条件和布尔操作

用于根据条件发射或变换Observables，或者对它们做布尔运算：

条件操作符

amb( ) — 给定多个Observable，只让第一个发射数据的Observable发射全部数据
defaultIfEmpty( ) — 发射来自原始Observable的数据，如果原始Observable没有发射数据，就发射一个默认数据
(rxjava-computation-expressions) doWhile( ) — 发射原始Observable的数据序列，然后重复发射这个序列直到不满足这个条件为止
(rxjava-computation-expressions) ifThen( ) — 只有当某个条件为真时才发射原始Observable的数据序列，否则发射一个空的或默认的序列
skipUntil( ) — 丢弃原始Observable发射的数据，直到第二个Observable发射了一个数据，然后发射原始Observable的剩余数据
skipWhile( ) — 丢弃原始Observable发射的数据，直到一个特定的条件为假，然后发射原始Observable剩余的数据
(rxjava-computation-expressions) switchCase( ) — 基于一个计算结果，发射一个指定Observable的数据序列
takeUntil( ) — 发射来自原始Observable的数据，直到第二个Observable发射了一个数据或一个通知
takeWhile( ) and takeWhileWithIndex( ) — 发射原始Observable的数据，直到一个特定的条件为真，然后跳过剩余的数据
(rxjava-computation-expressions) whileDo( ) — 如果条件为true，则发射源Observable数据序列，并且只要条件保持为true就重复发射此数据序列

布尔操作符

all( ) — 判断是否所有的数据项都满足某个条件
contains( ) — 判断Observable是否会发射一个指定的值
exists( ) and isEmpty( ) — 判断Observable是否发射了一个值
sequenceEqual( ) — 判断两个Observables发射的序列是否相等
