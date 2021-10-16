---
layout:     post
title:      golang 笔记9
subtitle:   结构体
date:       2021-10-09
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_65d.jpg
catalog: true
tags:
    - golang 学习笔记
---

# 结构体

## type

```go
// 自定义类型 [可以定义方法]
type myInt int

// 类型别名
type youInt = int

func TestType(t *testing.T) {
	var x myInt = 100  // package_name.myInt
	var y youInt = 100 // int
	fmt.Printf("%T\n", x)
	fmt.Printf("%T\n", y)
	/**
	=== RUN   TestType
	struct_demo.myInt
	int
	--- PASS: TestType (0.00s)
	PASS
	*/
}
```

## struct 结构体

#### 1. 函数内部使用

```go
func TestStruct1(t *testing.T) {
	var x struct {
		idNo string
		name string
	}
	x.idNo = "340001"
	x.name = "Jony"
	fmt.Println(x) // {340001 Jony}
}
```

#### 2. 结构体 + 自定义类型

```go

// 结构体 + 自定义类型
type Person struct {
	idNo       string
	name       string
	profession string
	age        int
	birthday   time.Time
}

type office struct {
	persons []Person
}

const YYYY_MM_DD = "2006-01-02"

func getTime(timeString string) time.Time {
	t, _ := time.Parse(YYYY_MM_DD, timeString)
	return t
}

// 结构体的方法
func (p Person) printPerson() {
	fmt.Println(p.idNo, p.name, p.profession, p.age, (&p.birthday).Format(YYYY_MM_DD))
}

func TestStruct2(t *testing.T) {
	var person1 Person
	person1.idNo = "341993001"
	person1.name = "morningcat"
	person1.profession = "golang工具人"
	person1.birthday = getTime("1993-04-05")
	person1.age = 28
	person1.printPerson() // 341993001 morningcat golang工具人 28 1993-04-05
}
```

#### 3. 值拷贝

```go
func changeName(p Person) { // 值拷贝
	p.name = "hello"
}

func changeName2(p *Person) { // 引用
	(*p).name = "hello"
	//p.name = "hello" // 语法糖 解析成上诉语句
}

func TestStruct3(t *testing.T) {
	var person Person
	person.name = "morningcat"

	changeName(person)
	fmt.Println(person.name) // morningcat

	changeName2(&person)
	fmt.Println(person.name) // hello
}

func TestStruct4(t *testing.T) {
	var person *Person = new(Person) // Person指针 （结构体指针）

	changeName2(person)
	fmt.Println(person.name)
}
```

#### 4. 初始化方式

```go
func TestStruct5(t *testing.T) {
	// 初始化
	var person3 = Person{idNo: "3419901", name: "mengzhang6", profession: "java", age: 28, birthday: getTime("1993-12-01")}
	person3.printPerson()

	// 格式化
	var person4 = Person{
		idNo:       "3419901",
		name:       "mengzhang6",
		profession: "java",
		age:        28,
		birthday:   getTime("1993-12-01"),
	}
	person4.printPerson()

	// 全量赋值 可省略key
	var person5 = Person{"3419901", "mengzhang6", "java", 28, getTime("1993-12-01")}
	person5.printPerson()
}
```

#### 5. 作用于特定类型的函数

```go

// 作用于特定类型的函数
// 任意自定义类型都可以加方法
// 只能给自己 package 中的`自定义类型`添加方法

func NewPerson(idNo, name string) *Person {
	return &Person{idNo: idNo, name: name}
}

func (p Person) GetPersonStr() string { //  值接收者
	return fmt.Sprintf(p.idNo, p.name, p.profession, p.age, (&p.birthday).Format(YYYY_MM_DD))
}

func TestStruct6(t *testing.T) {
	var p *Person = NewPerson("3419002", "morningcat")
	fmt.Println(p.GetPersonStr())
}

func (p *Person) ChangrPersonName(name string) { // 指针接收者
	p.name = name
}

func TestStruct7(t *testing.T) {
	var p = NewPerson("3419002", "morningcat")
	fmt.Println(p.name)
	p.ChangrPersonName("world")
	fmt.Println(p.name)
}

func (x myInt) helloInt() {
	fmt.Printf("Hello %d\n", x)
}

func TestStruct8(t *testing.T) {
	var x myInt = 12
	x.helloInt()
}

func TestStruct9(t *testing.T) {
	var x int32 = 12
	var y = int32(12) // 另一种定义方法
	fmt.Println(x, y)
}

```