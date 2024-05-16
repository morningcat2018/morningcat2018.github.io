---
layout:     post
title:      golang 笔记4
subtitle:   变量与常量和控制语句
date:       2021-10-04
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_65d.jpg
catalog: true
tags:
    - golang
---

# 变量与常量和控制语句

## 变量

```go
/**
定义在函数体外部为全局变量
*/
var str string = "hello world"
// 全局变量可以不使用

func fun1() {
    // 函数内部为局部变量

}
```

##  常量

```go
// 常量的声明与变量类似，只不过是使用 const 关键字。
// 常量不能用 := 语法声明。
const World = "世界"

const (
	// 将 1 左移 100 位来创建一个非常大的数字
	// 即这个数的二进制是 1 后面跟着 100 个 0
	Big = 1 << 100
	// 再往右移 99 位，即 Small = 1 << 1，或者说 Small = 2
	Small = Big >> 99
)
```

## if 结构

```go
func IfDemo(condition1, condition2 bool) {

    // Go 完全省略了结构中条件语句两侧的括号 使代码更加简洁
	if condition1 {
		// do something
	} else if condition2 {
		// do something else
	} else {
		// catch-all or default
	}
}
```

## switch 结构 （break continue return goto）

```go
func SwitchDemo(score int) string {
	str := ""
	switch score {
	case 1:
	case 2:
		str = "1 or 2"
	case 3:
		str = "3"
		// fallthrough
		// 使用 fallthrough 会强制执行后面的 case 语句，fallthrough 不会判断下一条 case 的表达式结果是否为 true
	case 4:
	case 5:
	case 6:
		str = "5,6,7"
	default:
		str = "other"
	}
	return str
}
```

## for (range) 结构

```go
func ForDemo() {
	arr := [5]string{"1", "2", "5", "3", "7"}
	for _, v := range arr {
		fmt.Println(v)
	}
}
```

## select 结构

```go
func main() {
	var c1, c2, c3 chan int
	var i1, i2 int
	select {
	case i1 = <-c1:
		fmt.Printf("received ", i1, " from c1\n")
	case c2 <- i2:
		fmt.Printf("sent ", i2, " to c2\n")
	case i3, ok := (<-c3): // same as: i3, ok := <-c3
		if ok {
			fmt.Printf("received ", i3, " from c3\n")
		} else {
			fmt.Printf("c3 is closed\n")
		}
	default:
		fmt.Printf("no communication\n")
	}
}
```
