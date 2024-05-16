---
layout:     post
title:      golang 笔记5
subtitle:   数组
date:       2021-10-05
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_65d.jpg
catalog: true
tags:
    - golang
---

# 数组


## 一维数组

#### 1. `数组` 必须 `给定容量`

```go
var arr [5]int // 数组 必须给定容量
for i := range arr {
    arr[i] = i
}
fmt.Printf("%T\n", arr)
```

#### 2. 数组初始化

```go
var arr = [5]string{"123", "Hello", "world", "好", "go"} // 全部初始化
for i := range arr {
    fmt.Println(arr[i])
}
fmt.Println("数组长度=", len(arr))
```

```go
arr := [5]string{"Hello", "world"} // 部分初始化
```

#### 3. 自动推断长度

```go
arr := [...]string{"a", "b", "c", "d"} // 自动推断长度
// 即
// var arr [4]string = [...]string{"a", "b", "c", "d"}
```

#### 4. 根据索引位置赋值

```go
arr := [5]string{1: "111", 4: "444"} // 根据索引位置赋值

for i, v := range arr {
    fmt.Println("下标：", i, "值：", v)
}
```

#### 5. `[3]int` 和 `[4]int` 不是同一种类型

```go
func fun1() [3]int {
    // var a [3]int
    // var a [4]int
    // a = b  不能这样赋值 因为 [3]int 和 [4]int 并不是同一种类型

	arr := [3]int{1, 2, 3}
	return arr
}
```

#### 6. 字符串分割

```go
func fun1() {
	s1 := "hello world This is golang"
	arr := strings.Split(s1, " ")
	fmt.Println(arr)
	fmt.Println(len(arr))
}
```

#### 7. `for i` 与 `for-range`

```go
func fun1() {
	var arr1 [5]int
	for i := 0; i < len(arr1); i++ {
		arr1[i] = i * 7
		fmt.Printf("Array at index %d is %d\n", i, arr1[i])
	}
}

func fun2() {
	var arr1 [5]int
	for i, _ := range arr1 {
		arr1[i] = i * 7
		fmt.Printf("Array at index %d is %d\n", i, arr1[i])
	}
}
```

#### 8. 数组是`值类型`

> go文件以 _test 结尾，函数名以 Test 开头，入参为 *testing.T 类型

```go
// arrat_test.go
import (
	"fmt"
	"testing"
)

func TestArrayDemo(t *testing.T) {
	// 数组是值类型 赋值和传参时会复制整个数组 因此改变副本的值不会改变本身的值
    // Go 语言中的数组是一种 值类型（不像 C/C++ 中是指向首元素的指针）
	arr := [5]int{1, 2, 3, 4, 5}
	var arr2 [5]int
	arr2 = arr
	arr2[1] = 200
	fmt.Println(arr)
	fmt.Println(arr2)
}
```

#### 9. `[5]int`(副本) 与 `*[5]int`(引用)

```go
var arr1 = new([5]int)   // arr1 类型 *[5]int  引用类型（即指针）
var arr2 [5]int          // arr2 类型  [5]int

arr3 := *arr1            // arr1 的副本
arr4 := arr2             // arr2 的副本

arr5 *[5]int = arr1      // arr1 的引用
var arr6 *[5]int = &arr2 // arr2 的引用
```

```go
// arrat_test.go
// 完整示例
func TestArray2Demo4(t *testing.T) {
	var arr1 = new([5]int) // arr1 的类型是 *[5]int
	var arr2 [5]int        // arr2 的类型是 [5]int
	for i, _ := range arr1 {
		arr1[i] = i
		arr2[i] = i
	}
	arr3 := *arr1
	arr3[2] = 300
	fmt.Println(arr3[2])
	fmt.Println(arr1[2])

	arr4 := arr2
	arr4[2] = 400
	fmt.Println(arr4[2])
	fmt.Println(arr2[2])

	var arr5 *[5]int = arr1
	arr5[2] = 500 // 会改变值
	fmt.Println(arr5[2])
	fmt.Println(arr1[2])

	var arr6 *[5]int = &arr2
	arr6[2] = 600 // 会改变值
	fmt.Println(arr6[2])
	fmt.Println(arr2[2])
}
```

> `引用和副本`

```go
func TestArray2(t *testing.T) {
	var array [3]int = [...]int{1, 2, 3}
	f(array) // passes a copy of ar
	fmt.Println(array[0])
	fp(&array) // passes a pointer to ar
	fmt.Println(array[0])
}
func f(a [3]int) { // 副本
	a[0] = 100
}
func fp(a *[3]int) { // 引用
	a[0] = 100
}
```

#### 10. `*[5]int`(引用) 和 `*[]int`(切片)

```go
/*
把一个大数组传递给函数会消耗很多内存[副本碎片]。有两种方法可以避免这种现象：
1. 传递数组的指针
2. 使用数组的切片
*/
func TestArray(t *testing.T) {
	var array1 = new([5]int)    // array1 的类型是 *[5]int
	var array2 [5]int           // array2 的类型是 [5]int
	var slice1 = make([]int, 5) // slice1 的类型是 *[]int
	for i, _ := range array1 {
		array1[i] = i + 1
		array2[i] = i + 1
		slice1[i] = i + 1
	}
	testPointer2(array1)  // 传入引用
	testPointer2(&array2) // 传入引用
	testPointer(slice1)   // 传入切片
}

func testPointer(slice []int) { // 切片
	slice[1] = 200
	print(slice)
}

func testPointer2(array *[5]int) { // 引用
	array[1] = 200
	print2(*array)
}

func print(slice []int) {
	for i, _ := range slice {
		fmt.Printf("%d\t", slice[i])
	}
	fmt.Println()
}

func print2(array [5]int) {
	for i, _ := range array {
		fmt.Printf("%d\t", array[i])
	}
	fmt.Println()
}
/*
=== RUN   TestArray
1	200	3	4	5
1	200	3	4	5
1	200	3	4	5
--- PASS: TestArray (0.00s)
PASS
 */
```

