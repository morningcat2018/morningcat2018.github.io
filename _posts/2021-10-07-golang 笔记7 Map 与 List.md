---
layout:     post
title:      golang 笔记7
subtitle:   Map 与 List
date:       2021-10-07
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_65d.jpg
catalog: true
tags:
    - golang 学习笔记
---

# Map 与 List

## map

#### 1. map(string,int)

```go

func TestMap1(t *testing.T) {
	// 定义 map(string,int)
	var myMap map[string]int = make(map[string]int, 10)

	// 赋值
	myMap["x"] = 1
	myMap["y"] = 3
	myMap["z"] = 5
	fmt.Println(myMap)

	// 取值
	v, ok := myMap["zs"]
	if !ok {
		fmt.Println("not found this key")
	} else {
		fmt.Println(v)
	}

	// 遍历
	for k, v := range myMap {
		fmt.Println(k, v)
	}

	// 删除
	delete(myMap, "y")
	fmt.Println(myMap)
	delete(myMap, "zs") // 删除不存在的key，不做操作
}
/*
=== RUN   TestMap1
map[x:1 y:3 z:5]
not found this key
y 3
x 1
z 5
map[x:1 z:5]
map[x:1 z:5]
--- PASS: TestMap1 (0.00s)
PASS
 */
```

#### 2. map(string,int) 数组

```go
func TestMap2(t *testing.T) {
	// 定义 map(string,int) 数组切片，容量自动扩容
	var s1 = make([]map[string]int, 10)

	// 索引0位置赋值 map1
	s1[0] = make(map[string]int)
	s1[0]["Hello"] = 123
	fmt.Println(s1)

	//s1[1]["Hello"] = 123 // error : not make map
}
```

#### 3. map(string,[]int)

```go
func TestMap3(t *testing.T) {
	// 定义 map(string,[]int)
	var s2 = make(map[string][]int, 10)
	s2["a"] = []int{1, 2, 3}
	fmt.Println(s2)
}
```

## list

#### 1. 基础

```go

import (
	"container/list"
	"fmt"
	"reflect"
	"testing"
)

func TestList1(t *testing.T) {
	var list *list.List = list.New()

	// 插入到尾部
	list.PushBack(1)
	list.PushBack(2)
	list.PushBack(3)
	// 插入到头部
	list.PushFront(4)
	list.PushFront("4")
	list.PushFront(5)
	list.PushFront(5)
	list.PushFront("5")
	list.PushFront("5")
	list.PushFront(6)
	// 遍历
	printList(list)

	// 删除
	deleteFirstValueIsFour(list)
	printList(list)

	deleteFirstValueIsFive(list)
	printList(list)
}

func deleteFirstValueIsFive(list *list.List) {
	isDelete := true
	for p := list.Front(); p != nil; p = p.Next() {
		if reflect.TypeOf(p.Value).Name() != "int" {
			continue
		}
		if isDelete && p.Value.(int) == 5 {
			list.Remove(p)
			isDelete = false
		}
	}

}

func deleteFirstValueIsFour(list *list.List) {
	for p := list.Front(); p != nil; p = p.Next() {
		switch p.Value.(type) {
		case int:
			if p.Value.(int) == 4 {
				list.Remove(p)
			}
			continue
		case string:
		case []interface{}:
		default:
			continue
		}
	}
}

func printList(list *list.List) {
	fmt.Println("\n**** list info *****")
	fmt.Println("len=", list.Len())
	// 遍历
	for p := list.Front(); p != nil; p = p.Next() {
		fmt.Print(p.Value, "(", reflect.TypeOf(p.Value), ") ")
	}
	fmt.Println("\n********************")
}
/**
=== RUN   TestList1

**** list info *****
len= 10
6(int) 5(string) 5(string) 5(int) 5(int) 4(string) 4(int) 1(int) 2(int) 3(int)
********************

**** list info *****
len= 9
6(int) 5(string) 5(string) 5(int) 5(int) 4(string) 1(int) 2(int) 3(int)
********************

**** list info *****
len= 8
6(int) 5(string) 5(string) 5(int) 4(string) 1(int) 2(int) 3(int)
********************
--- PASS: TestList1 (0.00s)
PASS
 */
```

#### 2.InsertAfter InsertBefore MoveToBack MoveToFront

```go
func TestList2(t *testing.T) {
	var list *list.List = list.New()

	// 插入到尾部
	list.PushBack(1)
	list.PushBack(2)
	list.PushBack(3)
	list.PushBack(4)
	list.PushBack(5)
	// 遍历
	printList(list)

	list.InsertAfter(200, list.Front().Next())
	list.InsertBefore(100, list.Front().Next())
	printList(list)

	//
	list.MoveToBack(list.Front())
	list.MoveToFront(list.Back().Prev())
	printList(list)
}
/*
=== RUN   TestList2

**** list info *****
len= 5
1(int) 2(int) 3(int) 4(int) 5(int) 
********************

**** list info *****
len= 7
1(int) 100(int) 2(int) 200(int) 3(int) 4(int) 5(int) 
********************

**** list info *****
len= 7
5(int) 100(int) 2(int) 200(int) 3(int) 4(int) 1(int) 
********************
--- PASS: TestList2 (0.00s)
PASS
 */
```