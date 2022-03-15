---
layout:     post
title:      golang 笔记3
subtitle:   基本类型
date:       2021-10-03
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_65d.jpg
catalog: true
tags:
    - golang 学习笔记
---

# 基本类型

## 定义局部变量的几种方式

```go
func BaseType() {
	// 定义在函数内的为局部变量
	var name string = "morningcat" // 完整
	var name2 = "morningcat" // 类型推断
	name3 := "morningcat" // 简短声明 [只能声明局部变量]

    // 定义在函数内的变量，声明后必须使用，否则通过不了编译
}
```

## 字符串

```go
var name string = "morningcat"

// 多行文本
var str2 string = `
    枯蓬老树昏鸦
    小桥流水人家
    古道
`
fmt.Println(len(str2))
```

```go
// 字符串拼接
s1 := "hello "
s2 := "world"
s3 := s1 + s2

s4 := fmt.Sprintf("%s %s ，你好 世界", s1, s2)
```

```go
// 字符串遍历
s1 := "hello世界"
for index, value := range s1 {
    fmt.Printf("下标：%d => %c\n", index, value)
}
```

## 数值

#### 1. int

```go
// int, uint 和 uintptr 在 32 位系统上通常为 32 位宽，在 64 位系统上则为 64 位宽
// 当你需要一个整数值时应使用 int 类型，除非你有特殊的理由使用固定大小或无符号的整数类型。
var count int = 5

var id = 9 // 类型推断为 int
```

各种表示方式：
```go
	var number int = 100
	fmt.Printf("%d\n", number)
	fmt.Printf("%b\n", number) // 二进制
	fmt.Printf("%o\n", number) // 八进制
	fmt.Printf("%x\n", number) // 十六进制 ff
	fmt.Printf("%X\n", number) // 十六进制

	a := 100
	b := 0b1001
	c := 072
	d := 0xFF
	fmt.Printf("%d\n", a)
	fmt.Printf("%d\n", b)
	fmt.Printf("%d\n", c)
	fmt.Printf("%d\n", d)
```

#### 2. 其他整型

```go
var id2 int8 = 0 // 8byte 范围：-128~127
var id3 int16 = -1
var id4 int32 = 1
var id5 int64 = 1
var id6 uint = 1 // 无符号数值
var id7 uint8 = 1
var id8 uint16 = 1
var id9 uint32 = 1
var id0 uint64 = 1
var id11 uintptr = 45
var di1 byte = 2 // uint8 的别名
var di2 rune = 8 // int32 的别名
```

#### 3. 浮点型

```go
var a = 0.31 // float64
b := 3.14    // float64

var c float32 = 0.1
var d float64 = 0.31
```

#### 4. 复数

```go
var mi1 complex64 = 3.7E+1 + 5.98E-2i // 实部和虚部都是 float32
var mi2 complex128 = 0.867 + 0.5i     // 实部和虚部都是 float64
```

## 布尔类型

```go
var isEat bool = true
var flag = false
gg := true
```

## 空值

```go
/**
    没有明确初始值的变量声明会被赋予它们的 [零值]。
    各种类型的零值是：
    数值类型为 0，
    布尔类型为 false，
    字符串为 ""（空字符串）。
*/
var in_v int
var f float64
var bo_v bool
var s string
fmt.Printf("%v %v %v %q\n", in_v, f, bo_v, s)
```