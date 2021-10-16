---
layout:     post
title:      关于 Java8 中的 SCFP 和 Stream 梳理
subtitle:   
date:       2019-11-12
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_637.jpg
catalog: true
tags:
    - java8
---

# 1. SCFP

- Supplier
- Consumer
- Function
- Predicate

## Supplier 提供者

```java
@FunctionalInterface
public interface Supplier<T> {

    T get();
}
```

#### Supplier 扩展

- BooleanSupplier 提供Boolean
- DoubleSupplier 提供Double
- IntSupplier 提供Int
- LongSupplier 提供Long

```java
@FunctionalInterface
public interface BooleanSupplier {

    boolean getAsBoolean();
}
```

```java
@FunctionalInterface
public interface DoubleSupplier {

    double getAsDouble();
}
```

```java
@FunctionalInterface
public interface IntSupplier {

    int getAsInt();
}
```

```java
@FunctionalInterface
public interface LongSupplier {

    long getAsLong();
}
```

#### Supplier 实践

```java
        Supplier<String> supplier = () -> {
            System.out.println("something running ...");
            return "run success";
        };

        System.out.println(supplier.get());

        // run result :
        // something run ...
        // run success
```

## Consumer 消费者


```java
@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

#### Consumer 拓展

- BiConsumer 同时消费两个入参
- DoubleConsumer
- IntConsumer
- LongConsumer
- ObjDoubleConsumer
- ObjIntConsumer
- ObjLongConsumer

```java
@FunctionalInterface
public interface BiConsumer<T, U> {

    void accept(T t, U u);

    default BiConsumer<T, U> andThen(BiConsumer<? super T, ? super U> after) {
        Objects.requireNonNull(after);

        return (l, r) -> {
            accept(l, r);
            after.accept(l, r);
        };
    }
}
```

```java
@FunctionalInterface
public interface DoubleConsumer {

    void accept(double value);

    default DoubleConsumer andThen(DoubleConsumer after) {
        Objects.requireNonNull(after);
        return (double t) -> { accept(t); after.accept(t); };
    }
}
```

```java
@FunctionalInterface
public interface ObjDoubleConsumer<T> {

    void accept(T t, double value);
}
```

#### Consumer 实践

```java

        Consumer<String> consumer = t -> {
            System.out.println("sayConsumer : " + t);
        };

        Consumer<String> anotherConsumer = t -> {
            System.out.println("anotherConsumer : " + t);
        };

        // 先运行自身，再 anotherConsumer
        consumer.andThen(anotherConsumer).accept("morningcat");

        // sayConsumer : morningcat
        // anotherConsumer : morningcat
```

```java
        BiConsumer<String, Long> biConsumer = (t, u) -> {
            System.out.println(t + " say : hello " + u);
        };

        // 先运行自身，再 anotherConsumer
        biConsumer.andThen((t, u) -> System.out.println(t + " say : hello hello " + u)).accept("morningcat", System.currentTimeMillis());

        // morningcat say : hello 1573525638320
        // morningcat say : hello hello 1573525638320
```

## Predicate 判断者

```java
@FunctionalInterface
public interface Predicate<T> {

    boolean test(T t);

    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}

```

#### Predicate 拓展

- BiPredicate 两个入参进行判断
- DoublePredicate
- IntPredicate
- LongPredicate

```java
@FunctionalInterface
public interface BiPredicate<T, U> {

    boolean test(T t, U u);

    default BiPredicate<T, U> and(BiPredicate<? super T, ? super U> other) {
        Objects.requireNonNull(other);
        return (T t, U u) -> test(t, u) && other.test(t, u);
    }

    default BiPredicate<T, U> negate() {
        return (T t, U u) -> !test(t, u);
    }

    default BiPredicate<T, U> or(BiPredicate<? super T, ? super U> other) {
        Objects.requireNonNull(other);
        return (T t, U u) -> test(t, u) || other.test(t, u);
    }
}
```

```java
@FunctionalInterface
public interface DoublePredicate {

    boolean test(double value);

    default DoublePredicate and(DoublePredicate other) {
        Objects.requireNonNull(other);
        return (value) -> test(value) && other.test(value);
    }

    default DoublePredicate negate() {
        return (value) -> !test(value);
    }

    default DoublePredicate or(DoublePredicate other) {
        Objects.requireNonNull(other);
        return (value) -> test(value) || other.test(value);
    }
}
```

#### Predicate 实践

```java

        Predicate<Integer> predicate = t -> t > 60;

        // 基础判断
        System.out.println(predicate.test(100));

        // 与操作
        System.out.println(predicate.and(t -> t > 80).test(100));

        // 或操作
        System.out.println(predicate.or(t -> t > 120).test(100));

        // 非操作
        System.out.println(predicate.negate().test(100));

        // static isEqual
        System.out.println(Predicate.isEqual("S").test("S"));

        // true
        // true
        // true
        // false
        // true
```

## Function 转换者

```java
@FunctionalInterface
public interface Function<T, R> {

    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}

```

#### Function 实践

```java
    @Getter
    @NoArgsConstructor
    @AllArgsConstructor
    @ToString
    class User {
        private String id;
        private String name;
    }

    public static void main(String[] args) {

        // String -> User
        Function<String, User> function = t -> new User(UUID.randomUUID().toString(), t);
        System.out.println(function.apply("morningcat"));

        // Long -> String -> User
        Function<Long, String> before = t -> String.valueOf(t);
        User user = function.compose(before).apply(2012211598L);
        System.out.println(user);

        // Long -> String -> User -> String
        Function<User, String> after = t -> t.getName();
        String name = function.compose(before).andThen(after).apply(2012211598L);
        System.out.println(name);
    }

    // User(id=559dff4b-ee38-4acd-b10c-83c7d22b4a35, name=morningcat)
    // User(id=5e6d2c63-73d1-40d9-80da-2cdf9dd7f125, name=2012211598)
    // 2012211598
```

#### Function 拓展 

- DoubleFunction
- DoubleToIntFunction
- DoubleToLongFunction
- IntFunction
- IntToDoubleFunction
- IntToLongFunction
- LongFunction
- LongToDoubleFunction
- LongToIntFunction
- ToDoubleFunction
- ToIntFunction
- ToLongFunction

- BiFunction
- ToDoubleBiFunction
- ToIntBiFunction
- ToLongBiFunction

```java
@FunctionalInterface
public interface DoubleFunction<R> {

    R apply(double value);
}

@FunctionalInterface
public interface DoubleToIntFunction {

    int applyAsInt(double value);
}

@FunctionalInterface
public interface DoubleToLongFunction {

    long applyAsLong(double value);
}

@FunctionalInterface
public interface ToDoubleFunction<T> {

    double applyAsDouble(T value);
}
```

```java
@FunctionalInterface
public interface BiFunction<T, U, R> {

    R apply(T t, U u);

    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t, U u) -> after.apply(apply(t, u));
    }
}

@FunctionalInterface
public interface ToDoubleBiFunction<T, U> {

    double applyAsDouble(T t, U u);
}
```

#### XxxOperator

- UnaryOperator
- DoubleUnaryOperator
- LongUnaryOperator
- IntUnaryOperator

- BinaryOperator
- DoubleBinaryOperator
- LongBinaryOperator
- IntBinaryOperator

```java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {

    static <T> UnaryOperator<T> identity() {
        return t -> t;
    }
}

@FunctionalInterface
public interface DoubleUnaryOperator {

    double applyAsDouble(double operand);

    default DoubleUnaryOperator compose(DoubleUnaryOperator before) {
        Objects.requireNonNull(before);
        return (double v) -> applyAsDouble(before.applyAsDouble(v));
    }

    default DoubleUnaryOperator andThen(DoubleUnaryOperator after) {
        Objects.requireNonNull(after);
        return (double t) -> after.applyAsDouble(applyAsDouble(t));
    }

    static DoubleUnaryOperator identity() {
        return t -> t;
    }
}
```

```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T,T,T> {

    public static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) <= 0 ? a : b;
    }

    public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
    }
}

@FunctionalInterface
public interface DoubleBinaryOperator {

    double applyAsDouble(double left, double right);
}
```

# 2. Stream API

## 基础

```java
public interface Stream<T> extends BaseStream<T, Stream<T>> {
    //...
}
```

```java
public interface BaseStream<T, S extends BaseStream<T, S>>
        extends AutoCloseable {

    Iterator<T> iterator();

    Spliterator<T> spliterator();

    // 是否是并行（重要）
    boolean isParallel();

    // 转换成串行
    S sequential();

    // 转换成并行
    S parallel();

    //
    S unordered();

    S onClose(Runnable closeHandler);

    void close();
}

```

## 创建

#### Stream.java

```java
    public static<T> Stream<T> empty() {
        return StreamSupport.stream(Spliterators.<T>emptySpliterator(), false);
    }

    public static<T> Stream<T> of(T t) {
        return StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
    }

    public static<T> Stream<T> of(T... values) {
        return Arrays.stream(values);
    }


    public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f) {
        Objects.requireNonNull(f);
        final Iterator<T> iterator = new Iterator<T>() {
            @SuppressWarnings("unchecked")
            T t = (T) Streams.NONE;

            @Override
            public boolean hasNext() {
                return true;
            }

            @Override
            public T next() {
                return t = (t == Streams.NONE) ? seed : f.apply(t);
            }
        };
        return StreamSupport.stream(Spliterators.spliteratorUnknownSize(
                iterator,
                Spliterator.ORDERED | Spliterator.IMMUTABLE), false);
    }

    public static<T> Stream<T> generate(Supplier<T> s) {
        Objects.requireNonNull(s);
        return StreamSupport.stream(
                new StreamSpliterators.InfiniteSupplyingSpliterator.OfRef<>(Long.MAX_VALUE, s), false);
    }

    public static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b) {
        Objects.requireNonNull(a);
        Objects.requireNonNull(b);

        @SuppressWarnings("unchecked")
        Spliterator<T> split = new Streams.ConcatSpliterator.OfRef<>(
                (Spliterator<T>) a.spliterator(), (Spliterator<T>) b.spliterator());
        Stream<T> stream = StreamSupport.stream(split, a.isParallel() || b.isParallel());
        return stream.onClose(Streams.composedClose(a, b));
    }

```

#### Collection.java

```java
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }

    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
```

#### Arrays.java

```java
    public static <T> Stream<T> stream(T[] array) {
        return stream(array, 0, array.length);
    }

    public static <T> Stream<T> stream(T[] array, int startInclusive, int endExclusive) {
        return StreamSupport.stream(spliterator(array, startInclusive, endExclusive), false);
    }
```


## 中间操作

#### 映射(转换)

```java
   <R> Stream<R> map(Function<? super T, ? extends R> mapper);

   IntStream mapToInt(ToIntFunction<? super T> mapper);

   LongStream mapToLong(ToLongFunction<? super T> mapper);

   DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper);

   <R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper);

   IntStream flatMapToInt(Function<? super T, ? extends IntStream> mapper);

   LongStream flatMapToLong(Function<? super T, ? extends LongStream> mapper);

   DoubleStream flatMapToDouble(Function<? super T, ? extends DoubleStream> mapper);
```

#### 排序

```java
    Stream<T> sorted();

    Stream<T> sorted(Comparator<? super T> comparator);

    // 配合 Comparator.comparingXxx
    public static <T, U> Comparator<T> comparing(
            Function<? super T, ? extends U> keyExtractor,
            Comparator<? super U> keyComparator)
    {
        Objects.requireNonNull(keyExtractor);
        Objects.requireNonNull(keyComparator);
        return (Comparator<T> & Serializable)
            (c1, c2) -> keyComparator.compare(keyExtractor.apply(c1),
                                              keyExtractor.apply(c2));
    }
    public static <T, U extends Comparable<? super U>> Comparator<T> comparing(
            Function<? super T, ? extends U> keyExtractor)
    {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2));
    }

    public static <T> Comparator<T> comparingInt(ToIntFunction<? super T> keyExtractor) {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> Integer.compare(keyExtractor.applyAsInt(c1), keyExtractor.applyAsInt(c2));
    }
    public static <T> Comparator<T> comparingLong(ToLongFunction<? super T> keyExtractor) {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> Long.compare(keyExtractor.applyAsLong(c1), keyExtractor.applyAsLong(c2));
    }
    public static<T> Comparator<T> comparingDouble(ToDoubleFunction<? super T> keyExtractor) {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> Double.compare(keyExtractor.applyAsDouble(c1), keyExtractor.applyAsDouble(c2));
    }

    default Comparator<T> reversed() {
        return Collections.reverseOrder(this);
    }
    public static <T extends Comparable<? super T>> Comparator<T> reverseOrder() {
        return Collections.reverseOrder();
    }
    public static <T extends Comparable<? super T>> Comparator<T> naturalOrder() {
        return (Comparator<T>) Comparators.NaturalOrderComparator.INSTANCE;
    }

```

```java
        // 正序 -> 从小到大
        Stream<Student> stream3 = stream1.sorted(Comparator.comparingInt(Student::getAge));
        // 逆序
        Stream<Student> stream4 = stream1.sorted(Comparator.comparingInt(Student::getAge).reversed());
```

#### 切片

```java
    // 截取前N个元素
    Stream<T> limit(long maxSize);

    // 忽略前N个元素
    Stream<T> skip(long n);
```

#### 筛选

```java
    Stream<T> filter(Predicate<? super T> predicate);
```

#### 去重

```java
    Stream<T> distinct();
```

#### peek

```java
    Stream<T> peek(Consumer<? super T> action);

    // demo
    Stream.of(5, 9, 8).peek(t -> System.out.println("peek:" + t)).forEach(System.out::println);
    // peek:5
    // 5
    // peek:9
    // 9
    // peek:8
    // 8
```

## 终止操作

#### 遍历

```java
    void forEach(Consumer<? super T> action);

    void forEachOrdered(Consumer<? super T> action);
```

#### 查找


```java
    Optional<T> findFirst();

    Optional<T> findAny();

    long count();

    Optional<T> min(Comparator<? super T> comparator);

    Optional<T> max(Comparator<? super T> comparator);
```

#### 匹配


```java
    // 有符合条件的元素存在
    boolean anyMatch(Predicate<? super T> predicate);

    // 所有的元素全部符合条件
    boolean allMatch(Predicate<? super T> predicate);

    // 没有符合条件的元素
    boolean noneMatch(Predicate<? super T> predicate);
```



#### 归约


```java
    Optional<T> reduce(BinaryOperator<T> accumulator);

    T reduce(T identity, BinaryOperator<T> accumulator);

    <U> U reduce(U identity,
                 BiFunction<U, ? super T, U> accumulator,
                 BinaryOperator<U> combiner);

```

#### 转换

```java
    Object[] toArray();

    <A> A[] toArray(IntFunction<A[]> generator);
```

#### collect


```java
    <R, A> R collect(Collector<? super T, A, R> collector);

    <R> R collect(Supplier<R> supplier,
                  BiConsumer<R, ? super T> accumulator,
                  BiConsumer<R, R> combiner);
```

// Collectors

```java
https://blog.csdn.net/u013837825/article/details/87191393#44__collect_280
```

# 集合

## List

// ...

## Map

```java
    default V getOrDefault(Object key, V defaultValue) {
        V v;
        return (((v = get(key)) != null) || containsKey(key))
            ? v
            : defaultValue;
    }

    default void forEach(BiConsumer<? super K, ? super V> action) {
        Objects.requireNonNull(action);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
            action.accept(k, v);
        }
    }

    default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
        Objects.requireNonNull(function);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }

            // ise thrown from function is not a cme.
            v = function.apply(k, v);

            try {
                entry.setValue(v);
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
        }
    }

    default V putIfAbsent(K key, V value) {
        V v = get(key);
        if (v == null) {
            v = put(key, value);
        }

        return v;
    }

    default boolean remove(Object key, Object value) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, value) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        remove(key);
        return true;
    }

    default boolean replace(K key, V oldValue, V newValue) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, oldValue) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        put(key, newValue);
        return true;
    }

    default V replace(K key, V value) {
        V curValue;
        if (((curValue = get(key)) != null) || containsKey(key)) {
            curValue = put(key, value);
        }
        return curValue;
    }

    default V computeIfAbsent(K key,
            Function<? super K, ? extends V> mappingFunction) {
        Objects.requireNonNull(mappingFunction);
        V v;
        if ((v = get(key)) == null) {
            V newValue;
            if ((newValue = mappingFunction.apply(key)) != null) {
                put(key, newValue);
                return newValue;
            }
        }

        return v;
    }

    default V computeIfPresent(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        V oldValue;
        if ((oldValue = get(key)) != null) {
            V newValue = remappingFunction.apply(key, oldValue);
            if (newValue != null) {
                put(key, newValue);
                return newValue;
            } else {
                remove(key);
                return null;
            }
        } else {
            return null;
        }
    }

    default V compute(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        V oldValue = get(key);

        V newValue = remappingFunction.apply(key, oldValue);
        if (newValue == null) {
            // delete mapping
            if (oldValue != null || containsKey(key)) {
                // something to remove
                remove(key);
                return null;
            } else {
                // nothing to do. Leave things as they were.
                return null;
            }
        } else {
            // add or replace old mapping
            put(key, newValue);
            return newValue;
        }
    }

    default V merge(K key, V value,
            BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        Objects.requireNonNull(value);
        V oldValue = get(key);
        V newValue = (oldValue == null) ? value :
                   remappingFunction.apply(oldValue, value);
        if(newValue == null) {
            remove(key);
        } else {
            put(key, newValue);
        }
        return newValue;
    }
```

