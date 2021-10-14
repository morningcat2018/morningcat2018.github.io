---
layout:     post
title:      golang 笔记11
subtitle:   时间 time
date:       2021-10-11
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_65d.jpg
catalog: true
tags:
    - golang 学习笔记
---

# 时间 time


## 1. 时间格式化

```go
const DEFAULT_TIME_FORMAT = "2006-01-02 15:04:05"
const YYYY_MM_DD = "2006-01-02"

func printTime(t time.Time) {
	fmt.Println(t.Format(DEFAULT_TIME_FORMAT))
	year, week := t.ISOWeek()
	fmt.Printf("%4d年 第%02d周 %s\n", year, week, t.Weekday())
}

func TestTimeFormat(te *testing.T) {
	var t time.Time = time.Now()
	fmt.Println(t.String())
	fmt.Println(t.Format(time.RFC822))
	fmt.Printf("%4d-%02d-%02d %02d:%02d:%02d\n", t.Year(), t.Month(), t.Day(), t.Hour(), t.Minute(), t.Second())
	printTime(t)
}
```

## 2. 获取时间

```go
func TestGetTime(te *testing.T) {
	t, err := time.Parse(DEFAULT_TIME_FORMAT, "2021-05-01 10:14:25")
	if err != nil {
		fmt.Printf("An error occurred\n")
		return // exit the function on error
	}
	printTime(t)
}
func GetTime(timeString string) time.Time {
	t, err := time.Parse(YYYY_MM_DD, timeString)
	if err != nil {
		panic("时间格式有误")
	}
	return t
}

func TestGetTime2(te *testing.T) {
	location, err := time.LoadLocation("UTC")
	if err != nil {
		return
	}
	t := time.Date(2021, 10, 1, 9, 15, 0, 0, location)
	printTime(t)
}
```

## 3. 时间计算

```go
func TestTimeAdd(te *testing.T) {
	t := time.Now()
	var one_week time.Duration = 60 * 60 * 24 * 7 * 1e9 // must be in nanosec (1s = 1000000000 ns)
	one_week_after := t.Add(one_week)
	printTime(one_week_after)
}
```

## 4. 线程休眠

```go
// 休眠
func TestTimeSleep(t *testing.T) {
	printTime(time.Now())
	var second5 time.Duration = 5 * 1e9 // 5s
	time.Sleep(second5)
	printTime(time.Now())
}
```