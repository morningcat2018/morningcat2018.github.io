---
layout:     post
title:      golang 笔记2
subtitle:   Go 与 Java 语法上的区别
date:       2021-10-02
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_65d.jpg
catalog: true
tags:
    - golang 学习笔记
---

# golang 笔记2 Go 与 Java 语法上的区别


##  区别

1. 结尾无需使用 ; 

```go
strValue := "hello world"
fmt.Println(strValue)
```



2. 定义的局部变量必须使用

仅声明一个局部变量却不使用，无法通过编译

```go
func fun1() {
	str := "你好"
	// 编译报错
}
```

3. 变量类型在变量名后面

```go
var strValue string = "hello world"
```

4. 函数返回值在函数名后面

```go
func hello(name string) string {
	return "hello" + name
}
```

5. 多返回值

```go
// 函数可以返回任意数量的返回值
func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b)
}
```

6. if 判断无需括号

```go
if count >= 60 {
    fmt.Println("已及格")
}
```

7. 无需某个返回值，可用下划线替代

```go
// 字符串遍历
s1 := "hello世界"

for _, value := range s1 {
    fmt.Println(value)
}
```

99. 支持闭包

[GoLang 闭包实践](https://blog.csdn.net/u013837825/article/details/114649547)

100. 未完待续


