---
layout:     post
title:      golang 笔记13
subtitle:   interface
date:       2021-10-13
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_65d.jpg
catalog: true
tags:
    - golang 学习笔记
---

# golang 笔记13 interface

1. interface 基础

> 一个变量如果实现了接口中定义的所有方法 那么这个变量可以理解为这种接口类型的变量

```go
package interface_demo

import (
	"fmt"
	"testing"
)

// 	定义接口
type speaker interface {
	speak() // 只要实现了 speak 方法的变量都是 speaker 类型
}

type dog struct{ name string }
type cat struct{ age int }

func (d dog) speak() {
	fmt.Println(d.name, ":汪汪汪")
}
func (c cat) speak() {
	fmt.Println("喵喵喵", c.age)
}

// 使用接口
func call(sp speaker) {
	sp.speak()
}

// 调用接口方法
func TestInterfaceDemo1(t *testing.T) {
	var c1 cat
	c1.age = 12
	call(c1)

	var d1 dog
	d1.name = "dico"
	call(d1)

	var sp speaker
	sp = c1
	sp.speak()
}
```

2. 空接口

```go
// interface{} 空接口类型
type blank interface{} // 所有类型都实现了空接口 即所有类型都相当于这种接口类型的变量

// 空接口 作为入参
func BlankInterfaceDemo1(param interface{}) {
	fmt.Println(param)
}

// 空接口列表
func BlankInterfaceDemo2(params ...interface{}) {
	for i, v := range params {
		fmt.Println(i, v)
	}
}

// 空接口 作为 map 的 value
func BlankInterfaceDemo3() {
	var interMap map[string]interface{} = map[string]interface{}{}
	interMap["hello"] = 514
	interMap["world"] = "world"
	interMap["mc"] = time.Now()
	interMap["xx"] = 's'
	fmt.Println(interMap)
}
```

3. 类型断言

```go

/*
x.(T)
x : 类型为 interface{} 的变量
T : 表示断言 x 可能的类型
*/
func interface_demo4(param interface{}) {
	value, ok := param.(string)
	if !ok {
		fmt.Println("param not is string:", param)
		return
	}
	fmt.Printf("param is string: %s\n", value)
}

// 不建议使用
func interface_demo5(param interface{}) {
	switch t := param.(type) {
	case int:
		fmt.Println("is int -> ", t)
	case string:
		fmt.Println("is string -> ", t)
	case bool:
		fmt.Println("is bool -> ", t)
	case float64:
		fmt.Println("is float -> ", t)
		// ...
	}
}

func TestBlankInterfaceDemo4(t *testing.T) {
	interface_demo4("hello")
	interface_demo4(1024)
	interface_demo5("world")
	interface_demo5(1024)
	interface_demo5(true)
	interface_demo5(3.1415926)
}

```