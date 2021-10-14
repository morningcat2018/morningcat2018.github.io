---
layout:     post
title:      golang 笔记1
subtitle:   基础知识
date:       2021-10-01
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_65d.jpg
catalog: true
tags:
    - golang 学习笔记
---

# 基础知识

## 基础

Go is an open source programming language that makes it easy to build simple, reliable, and efficient software.

[golang官网 : golang.org/](https://golang.org/)

[golang github : github.com/golang/go](https://github.com/golang/go)

## 安装

https://golang.org/doc/install

```
➜  ~ go version
go version go1.16.2 darwin/amd64
```

## 配置

1. GOROOT

2. GOPATH

## golang 工作目录

```
～/go-workspace
    |
    | - bin
    |
    | - pkg
    |
    | - src
        |
        | - github.com
                |
                | - my-account [github 账户]
                |
                | - morningcat2018
                        |
                        | - my-project [工程名称]
                        |
                        | - studygo
```

## hello world

```go
/**
 * main.go
 */
package main

import "fmt"

func main() {
	fmt.Println("hello world")
}
```



## 附录：golang学习笔记与实践

- [morningcat2018/studygo](https://github.com/morningcat2018/studygo)

- [morningcat2018/book-manage](https://github.com/morningcat2018/book-manage)