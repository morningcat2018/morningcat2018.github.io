---
layout:     post
title:      CompletableFuture API 笔记
subtitle:   
date:       2021-08-30
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_637.jpg
catalog: true
tags:
    - java
---


## 1. 异步开启一个任务

- supplyAsync
    - Supplier<U> : T get()
    - 无入参，有返回值
- runAsync
    - Runnable : void run()
    - 无返回值



## 2. 任务后置处理

- thenApply
    - Function<? super T,? extends U> : R apply(T t)
    - 一个入参，有返回值
    - thenApplyAsync 异步执行后置任务
- thenAccept
    - Consumer<? super T> : void accept(T t)
    - 一个入参，无返回值
    - thenAcceptAsync 异步执行后置任务
- thenRun
    - Runnable : void run()
    - 无入参，无返回值
    - thenRunAsync 异步执行后置任务



## 3. 连接两个异步任务

- thenCompose
    - Function<? super T, ? extends CompletionStage<U>> :  R apply(T t)
    - 一个入参(前一个任务的结果)，有返回值（CompletionStage）


## 4. 合并两个异步任务

- thenCombine
    - CompletionStage<? extends U>
    - BiFunction<? super T,? super U,? extends V> : R apply(T t, U u)
    - thenCombineAsync
- thenAcceptBoth
    - CompletionStage<? extends U>
    - BiConsumer<? super T, ? super U>
    - thenAcceptBothAsync
- runAfterBoth
    - CompletionStage<? extends U>
    - Runnable
    - runAfterBothAsync


## 5. 获取最先执行完成的任务


- applyToEither
    - CompletionStage<? extends T>
    - Function<? super T, U> : R apply(T t)
    - applyToEitherAsync
- acceptEither
    - CompletionStage<? extends T>
    - Consumer<? super T>
    - acceptEitherAsync
- runAfterEither
    - CompletionStage<? extends T>
    - Runnable
    - runAfterEitherAsync


## 6. 异常捕获

- exceptionally
    - Function<Throwable, ? extends T> : R apply(T t)
    - 一个入参 Throwable，有返回值
- handle
    - BiFunction<? super T, Throwable, ? extends U> : R apply(T t, U u)
    - handleAsync
- whenComplete
    - BiConsumer<? super T, ? super Throwable> : void accept(T t, U u)
    - whenCompleteAsync


## 7. join

- join


---

简要案例

```java
    CompletableFuture.runAsync(() -> {
        System.out.println("helo");
    });
    CompletableFuture.supplyAsync(() -> "hello");

    CompletableFuture.supplyAsync(() -> "hello").thenApplyAsync(t -> t + "w");
    CompletableFuture.supplyAsync(() -> "hello").thenAcceptAsync(t -> {
    });
    CompletableFuture.supplyAsync(() -> "hello").thenRunAsync(() -> {
    });

    CompletableFuture.supplyAsync(() -> "hello").thenCombine(CompletableFuture.completedFuture("world"), (x, y) -> x + y);
    CompletableFuture.supplyAsync(() -> "hello").thenAcceptBoth(CompletableFuture.supplyAsync(() -> "world"), (x, y) -> {
    });
    CompletableFuture.supplyAsync(() -> "hello").runAfterBoth(CompletableFuture.supplyAsync(() -> "world"), () -> {
    });

    CompletableFuture.supplyAsync(() -> "hello").applyToEitherAsync(CompletableFuture.completedFuture("world"), x -> x);
    CompletableFuture.supplyAsync(() -> "hello").acceptEitherAsync(CompletableFuture.supplyAsync(() -> "world"), x -> {
    });
    CompletableFuture.supplyAsync(() -> "hello").runAfterEitherAsync(CompletableFuture.supplyAsync(() -> "world"), () -> {
    });

    CompletableFuture.supplyAsync(() -> "hello").exceptionally(e -> null);
    CompletableFuture.supplyAsync(() -> "hello").handle((t, e) -> {
        if (e != null) {
            e.printStackTrace();
            return e.getMessage();
        } else {
            return t + "world";
        }
    });
    CompletableFuture.supplyAsync(() -> "hello").whenComplete((t, e) -> {
        if (e != null) {
            e.printStackTrace();
        } else {
            System.out.println(t);
        }

    });
```