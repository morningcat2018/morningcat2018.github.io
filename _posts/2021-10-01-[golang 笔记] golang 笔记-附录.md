---
layout:     post
title:      golang 笔记14
subtitle:   搜集的示例代码
date:       2021-10-14
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_65d.jpg
catalog: true
tags:
    - golang 学习笔记
---

# 搜集的示例代码

## 1. 类型转换

```go
func TestBaseType(t *testing.T) {
	// 类型转换
	var x = -42
	var ux uint = uint(x)
	var fl float64 = float64(x)
	fmt.Println(x, fl, ux)
}
```

## 2. 计算md5值

```go
func TestMd5() {
	strValue := "hello world"
	hash := md5.New()
	_, error := hash.Write([]byte(strValue))
	if error != nil {
		fmt.Println(error)
		os.Exit(-1)
	}
	bytes := hash.Sum(nil)
	//fmt.Println(string(bytes))
	md5Value := fmt.Sprintf("%x", bytes)
	fmt.Println(md5Value)
}
```

## 3. 斐波那契

```go
package main

func main() {
	fmt.Println(fib(70))
}

var fibs [100]uint64

func fib(n uint) (res uint64) {
	if fibs[n] != 0 {
		res = fibs[n]
		return
	}
	if n < 2 {
		res = 1
	} else {
		res = fib(n-1) + fib(n-2)
	}
	fibs[n] = res
	return
}
```

## 4. 闭包封装

```go
package main

func main() {
	var once sync.Once
	once.Do(task1Wrapper(2, 3))
	once.Do(task1Wrapper("2", 3))
	once.Do(task1Wrapper("hello", "world"))
}

// http://topgoer.com/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/sync.html#synconce
// interface{} 可以表示任意类型
func task1(m, n interface{}) {
	fmt.Println(m, n)
}

func task1Wrapper(m, n interface{}) func() {
	// 使用闭包做一层封装，供那些只接收无参的方法使用
	return func() {
		task1(m, n)
	}
}
```

## 5. package 中的常识

```go
package main

import (
	. "fmt"                      // 使用 fmt 下的方法，可以忽略 `fmt.`
	_ "studygo/book_manage"      // 使用 _ ，可以不使用该包中的方法，且自动调用该包中的 init() 方法
	ind "studygo/interface_demo" // 导入的包起别名
)

// 初始化方法，会自动调用
func init() {
	Println("当前包初始化方法")
}

func main() {
	Println("Hello world")
	ind.BlankInterfaceDemo4()
}
```

## 6. 局部作用域

```go
func Test1(t *testing.T) {
	if v := math.Pow(2, 10); v > 1000 {
		fmt.Printf("2^10=%f  > 1000\n", v) // v的作用域只在当前 {} 内
	}

	var osName = ""
	switch os := runtime.GOOS; os {
	case "darwin":
		osName = "MacOs"
	case "linux":
		osName = "Linux"
	default:
		osName = os
	}
	fmt.Println(osName)
}
```

## 7. bytes.Buffer

```go
func Test2(t *testing.T) {
	var buffer1 bytes.Buffer // 类似 java 中的 StringBuilder
	buffer1.WriteByte('m')
	buffer1.WriteString("cat")
	fmt.Println(string(buffer1.Bytes()))

	var buffer2 *bytes.Buffer = new(bytes.Buffer)
	buffer2.Write(buffer1.Bytes())
	buffer2.WriteRune(48)
	fmt.Println(string(buffer2.Bytes()))

	var buffer3 *bytes.Buffer = bytes.NewBuffer(buffer2.Bytes())
	buffer3.WriteString(" hi")
	fmt.Println(string(buffer3.Bytes()))
}
```

## 8. json

```go
func Test3(t *testing.T) {
	// map -> json string
	var map1 = make(map[string]interface{})
	map1["name"] = "mc2018"
	map1["age"] = 28
	map1["hobby"] = [...]string{"吃", "玩", "看动漫", "读书"}
	bytes, err := json.Marshal(map1)
	if err != nil {
		return
	}
	fmt.Println(string(bytes))

	// obj -> json string
	type Person struct {
		Name  string   `json:"姓名"`
		Age   uint8    `json:"年龄"`
		Hobby []string `json:"爱好"`
	}
	p1 := Person{"mc2018", 28, []string{"read", "eat"}}
	bytes2, err2 := json.Marshal(p1)
	if err2 != nil {
		return
	}
	fmt.Println(string(bytes2))
}
```

## 9. reflect

```go
func Test4(t *testing.T) {
	var x = 10
	v := reflect.ValueOf(x)
	fmt.Println(v.Kind())
	fmt.Println(v.Kind() == reflect.Int)

	w := reflect.TypeOf(x)
	fmt.Println(w.Name())
	fmt.Println(w.Name() == "int")
}
```

## 10. sort

```go
func Test5(t *testing.T) {
	var ints = [...]int{7, 3, 9, 2, 7, 1, 0, 45, 6}
	a := sort.IntSlice(ints[:])
	sort.Sort(a)
	fmt.Println(a)
}
```