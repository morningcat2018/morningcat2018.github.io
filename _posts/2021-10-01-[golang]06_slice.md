---
layout:     post
title:      golang 笔记6
subtitle:   切片
date:       2021-10-06
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_65d.jpg
catalog: true
tags:
    - golang
---

# 切片

## 1. make 切片


> 切片是一个`引用类型`
> 切片是一个`长度可变的数组`
> 因为切片是引用，所以它们不需要使用额外的内存并且比使用数组`更有效率`，所以在 Go 代码中 切片比数组更常用
> 声明切片的格式是： var identifierName []type（`不需要说明长度`）


```go
func TestSlice1(t *testing.T) {
	var slice []int = make([]int, 10)
	for i := 0; i < len(slice); i++ {
		slice[i] = i
	}
	printIntSlice("slice1", slice)
}

func printIntSlice(sliceName string, slice []int) {
	fmt.Println(slice)
	fmt.Printf("%s ->len=%d cap=%d\n", sliceName, len(slice), cap(slice)) // 长度  容量
}
```

```go
func TestSlice3(t *testing.T) {
	var slice []int = make([]int, 5, 10) // 长度为 5， 容量为 10
	for i := 0; i < len(slice); i++ {
		slice[i] = i
	}
	printIntSlice("slice", slice)
}
/*
=== RUN   TestSlice3
[0 1 2 3 4]
slice ->len=5 cap=10
--- PASS: TestSlice3 (0.00s)
PASS
 */
```

## 2. 从`数组`切割出切片

> 切片（Slice）是一个拥有相同类型元素的`可变长度`的序列
> 基于数组类型做的一层封装
> `支持自动扩容`
> 它的内部结构包含地址、长度和容量

```go
func TestSlice2(t *testing.T) {
	array := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	slice1 := array[1:5] // 1 2 3 4
	printIntSlice("slice1", slice1)
	slice2 := array[:5]  // 0 1 2 3 4
	printIntSlice("slice2", slice2)
	slice3 := array[5:]  // 5 6 7 8 9
	printIntSlice("slice3", slice3)
	slice4 := array[:]   // 0 1 2 3 4 5 6 7 8 9
	printIntSlice("slice4", slice4)

	// 切片再切片
	slice5 := slice1[1:3] // 1 2 3 4 -> 2 3
	printIntSlice("slice5", slice5)
}
/**
=== RUN   TestSlice2
[1 2 3 4]
slice1 ->len=4 cap=9
[0 1 2 3 4]
slice2 ->len=5 cap=10
[5 6 7 8 9]
slice3 ->len=5 cap=5
[0 1 2 3 4 5 6 7 8 9]
slice4 ->len=10 cap=10
[2 3]
slice5 ->len=2 cap=8
--- PASS: TestSlice2 (0.00s)
PASS
 */
```

> `容量`：底层数组从切片的第一个位置到最后位置的数量

## 3. 切片只是一个引用 `真正的数据`都保存在`底层数组`中

```go
func TestSlice(t *testing.T) {
	var arr []int // nil 值的切片没有底层数组 长度和容量都是0
	fmt.Println(arr == nil) // true
}
```

## 4. 遍历方式

```go
func TestSlice(t *testing.T) {
	var arr []int = make([]int, 5, 10)
	for i := 0; i < len(arr); i++ {
		arr[i] = i
	}
	for _, v := range arr {
		fmt.Println(v)
	}
}
```

## 5. append 追加元素

```go
func TestSliceAppend(t *testing.T) {
	slice := []int{1, 3, 5}
	printIntSlice("slice", slice)
	slice = append(slice, 7) // 扩容策略 slice.go row:144
	printIntSlice("slice", slice)
	//s1[4] = 9
	slice = append(slice, 9)
	printIntSlice("slice", slice)
}
/*
=== RUN   TestSliceAppend
[1 3 5]
slice ->len=3 cap=3
[1 3 5 7]
slice ->len=4 cap=6
[1 3 5 7 9]
slice ->len=5 cap=6
--- PASS: TestSliceAppend (0.00s)
PASS
*/
```


```go
func TestSliceAppend2(t *testing.T) {
	var slice []int = make([]int, 5, 10)
	for i := 0; i < len(slice); i++ {
		slice[i] = i*2 + 1
	}
	printIntSlice("slice", slice)

	slice = append(slice, 11, 13, 15, 17, 19)
	printIntSlice("slice", slice)

	slice = append(slice, 21)
	printIntSlice("slice", slice)

	ss := []int{2, 4, 6}
	slice = append(slice, ss...)
	printIntSlice("slice", slice)
}
/*
=== RUN   TestSliceAppend2
[1 3 5 7 9]
slice ->len=5 cap=10
[1 3 5 7 9 11 13 15 17 19]
slice ->len=10 cap=10
[1 3 5 7 9 11 13 15 17 19 21]
slice ->len=11 cap=20
[1 3 5 7 9 11 13 15 17 19 21 2 4 6]
slice ->len=14 cap=20
--- PASS: TestSliceAppend2 (0.00s)
PASS
*/
```

> ... 表示将切片`分解`成单个元素

```go
func TestSliceAppend3(t *testing.T) {
	var slice []int = make([]int, 3)

	ss := []int{2, 4, 6}
	slice = append(slice, ss...) // ... 表示将ss分解成单个元素
	printIntSlice("slice", slice)
}

```

## 6. 删除


```go
func TestSliceDelete(t *testing.T) {
	a1 := [...]int{1, 2, 3, 4, 5, 6, 7, 8, 9}
	s1 := a1[:]
	fmt.Println(s1) // 删除前

	s1 = append(s1[:1], s1[2:]...) // 切片删除索引1的元素
	fmt.Println(s1)                // 删除后
	fmt.Println(a1)
}
/**
=== RUN   TestSliceDelete
[1 2 3 4 5 6 7 8 9]
[1 3 4 5 6 7 8 9]
[1 3 4 5 6 7 8 9 9]
--- PASS: TestSliceDelete (0.00s)
PASS
 */
```

## 7. 深度拷贝

```go
func TestSliceCopy(t *testing.T) {
	a1 := []int{1, 3, 5}
	a2 := a1 // 赋值 浅度拷贝
	var a3 = make([]int, 3)
	copy(a3, a1) // copy 深度拷贝
	a1[0] = 100
	fmt.Println(a1, a2, a3)
}
/**
=== RUN   TestSliceCopy
[100 3 5] [100 3 5] [1 3 5]
--- PASS: TestSliceCopy (0.00s)
PASS
 */
```

