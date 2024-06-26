---
layout:     post
title:      golang 笔记8
subtitle:   函数
date:       2021-10-08
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_65d.jpg
catalog: true
tags:
    - golang
---

# 函数


## 基本函数

#### 1. 函数基础

> 格式 ： `func` 函数名(入参名 入参类型, ...) 返回参数类型 ... { // 函数体 }

> 函数首字母大写为公开函数，可以通过`包名.函数名`的方式调用
> 首字母小写为私有函数，只能在`当前包`内使用

```go
func Add(x, y int) int { // 与 Add(x int, y int) 等效
	var sum = x + y
	return sum
}
```

#### 2. 命名返回值

> Go 的返回值可被命名，它们会被视作定义在函数顶部的变量

```go
func split(sum int) (x, y int) {
	x = sum * 4 / 9.0
	y = sum - x
	// 没有参数的 return 语句返回已命名的返回值 : 直接返回。
	return
}

func Add2(x, y int) (sum int) {
	sum = x + y
	return // 不写返回值也可 因为返回值已经被命名
}
```

#### 3. 多返回值

> 函数可以返回任意数量的返回值

```go

func swap(x, y string) (string, string) {
	return y, x
}

func Add3(x, y int) (int, error) {
	var sum = x + y
	return sum, nil
}

func Add4(x, y int) (sum int, err error) {
	sum = x + y
	err = nil
	return
}
```

#### 4. 可变长参数

> 可变长参数 必须放在参数列表的最后

```go
func Add5(arg1 string, agr2 string, slice ...int) {
	fmt.Println(arg1, agr2)

	// slice 是一个切片类型 -> []int
	sum := 0
	for _, v := range slice {
		sum += v
	}
	fmt.Println(sum)
}

```

## 匿名函数

```go
func anonymityFun() {
	x := 1
	
	var xFun = func() {
		x++
	}
	xFun() // 调用

	// 匿名函数声明和调用
	func() {
		x++
	}()
}
```

## defer

> defer 类似 java 中的 finally 语句；在 return 之前执行
> defer 多用于函数结束之前释放资源 （文件句柄 数据库连接 socket连接...）

```go
func TestDefer(t *testing.T) {
	defer fmt.Println("defer 语句")
	fmt.Println("Hello world")
}
/*
=== RUN   TestDefer
Hello world
defer 语句
--- PASS: TestDefer (0.00s)
PASS
 */
```

#### 多个 defer 语句，会按照先进后出顺序执行【类似栈的操作，入栈 出栈】

```go
func TestDefer2(t *testing.T) {
	defer f1(1)
	fmt.Println("Hello defer 1")
	defer f1(2)
	fmt.Println("Hello defer 2")
	defer f1(3)
	fmt.Println("Hello defer 3")
}

func f1(x int) {
	fmt.Println(x)
}

/**
=== RUN   TestDefer2
Hello defer 1
Hello defer 2
Hello defer 3
3
2
1
--- PASS: TestDefer2 (0.00s)
PASS
 */
```

#### defer 语句中已经无法对函数内的变量进行改动了

```go

func TestDefer3(t *testing.T) {
	fmt.Println(f2())
	fmt.Println(f3())
	fmt.Println(f4())
	fmt.Println(f5())
}

func f2() int {
	x := 5
	defer func() {
		x++
	}()
	return x
}

func f3() (x int) { // 返回6 （注意：此处返回值已命名）
	defer func() {
		x++ // 第二步 x=x+1
	}()
	return 5 // 	第一步 x=5
} // 第三步 执行 RET 指令

func f4() (y int) {
	x := 5
	defer func() {
		x++
	}()
	return x
}

func f5() (x int) {
	defer func(x int) {
		x++
	}(x)
	return 5
}
```

## 高阶函数

> 高阶函数 函数可以作为参数和返回值
> 面向函数编程

#### 1. 函数赋值给变量

```go
func fc0()             {}
func fc1() string      { return "golang" }
func fc2(x int) string { return "golang" }

func TestClosure(t *testing.T) {
	a := fc0
	fmt.Printf("%T\n", a)
	b := fc1
	fmt.Printf("%T\n", b)
	c := fc2
	fmt.Printf("%T\n", c)
}
/**
=== RUN   TestClosure
func()
func() string
func(int) string
--- PASS: TestClosure (0.00s)
PASS
 */
```

#### 2. 函数作为入参

```go
func fc3(fun1 func()) {
	fun1()
}

func TestClosure2(t *testing.T) {
	a := func() {
		fmt.Println("TestClosure2")
	}

	fc3(a)
}

// 有入参

func fc4(fun2 func(int)) {
	fun2(10)
}

func TestClosure3(t *testing.T) {
	a := func(x int) {
		fmt.Println(x * 2)
	}
	fc4(a)
}

// 有出参

func fc5(x func(int) int) {
	fmt.Println(x(50))
}

func TestClosure4(t *testing.T) {
	a := func(x int) (y int) {
		y = x * 2
		return
	}
	fc5(a)
}
```

#### 3. 函数作为出参

```go
func fc6(x func(int, int), m, n int) func() {
	ret := func() {
		x(m, n)
	}
	return ret
}

func TestClosure5(t *testing.T) {
	var k1 = func(x, y int) {
		fmt.Println(x + y)
	}
	k2 := fc6(k1, 3, 4)
	k2()
}
```

#### 4. 匿名函数

```go
func TestClosure7(t *testing.T) {
	// 函数内部无法声明有名称的函数
	// 可以使用匿名函数
	var f1 = func(x int, y int) {
		fmt.Println(x + y)
	}
	f1(2, 3)

	// 声明加调用
	func(x int, y int) {
		fmt.Println(x + y)
	}(4, 5)
}
```

#### 5. 闭包

> 闭包 = 函数 + 外部变量的引用
> 1. 函数作为出参
> 2. 函数包含外部作用域的变量

```go
func addN(n int) func(int) int {
	return func(y int) int {
		return n + y
	}
}

func TestClosure6(t *testing.T) {
	var k3 = addN(100) // 得到一个与100相加的函数
	var sum = k3(245)
	fmt.Println(sum)
}
```

更多案例

```go

func addN(n int) func(x int) int {
	return func(x int) int {
		return x + n
	}
}

func addOrSubN(n int) (func(y int) int, func(y int) int) {
	addFunc := func(x int) int {
		return x + n
	}
	subFunc := func(x int) int {
		return x - n
	}
	return addFunc, subFunc
}

func TestClosure(t *testing.T) {
	addFunc1 := addN(10)
	fmt.Printf("%d\n", addFunc1(3))
	fmt.Printf("%d\n", addFunc1(7))

	addFunc, subFunc := addOrSubN(10)
	fmt.Printf("%d\n", addFunc(30))
	fmt.Printf("%d\n", subFunc(30))
}

func cal(base int) (func(y int) int, func(y int) int) {
	addFunc := func(x int) int {
		base += x // 共享变量
		return base
	}
	subFunc := func(x int) int {
		base -= x
		return base
	}
	return addFunc, subFunc
}

func cal2(base int) (func(y int) int, func(y int) int) {
	addFunc := func(x int) int {
		return base + x
	}
	subFunc := func(x int) int {
		return base - x
	}
	return addFunc, subFunc
}

func TestClosure2(t *testing.T) {
	add, sub := cal2(10)
	fmt.Println(add(2), sub(3))
	fmt.Println(add(3), sub(4))
}
```