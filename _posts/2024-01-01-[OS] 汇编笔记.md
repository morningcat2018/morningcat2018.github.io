---
layout:     post
title:      x86 汇编基础知识笔记
subtitle:   
date:       2024-01-01
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - OS
---

在 [linux内核分析,源码分析 - 中国科学技术大学软件学院MOOC](https://www.bilibili.com/video/BV1GJ411g7Gn/)学习梳理得到的笔记

- b 后缀,表示为 8bit
- w 后缀,表示为 16bit
- l 后缀,表示为 32bit
- q 后缀,表示为 64bit

## x86寄存器

![image.png](/png/x86.png)

rax rbx rcx rdx ... 表示64bit寄存器


## MOV 指令

1. register mode (寄存器寻址)

```
movel %eax, %edx
```

表示将 eax 寄存器里的内容放到 edx 里; 相等于 edx = eax;

2. immediate (立即寻址)

```
movel $0x123, %edx
```

将数值 0x123 放到寄存器 edx 里; 相当于 edx = 0x123;

3. direct  (直接寻址)

```
movel 0x123, %edx
```

将内存地址为0x123位置的内存数据放到寄存器 edx 里; 相当于 edx = *(int32_t *)0x123;

4. indirect (间接寻址)

```
movel (%ebx), %edx
```

将寄存器ebx里的值指向的内存地址,内存中所包含的数据放到寄存器 edx 里; 相当于 edx = *(int32_t *)ebx;

5. displaced (变址寻址)

```
movel 4(%ebx), %edx
```
将寄存器ebx里的值指向的内存地址 +4 所指向的内存中的数据,放到 寄存器 edx 里;
相当于 `edx = *(int32_t *)(ebx + 4);` 这里的 4 为 4byte 相当于 32bit


备注:
- 以 % 开头的寄存器标识符
- 立即数是以 $ 开头的数值
- 直接寻址:直接访问一个指定的内存地址的数据
- 间接寻址:将寄存器的值作为一个内存地址来访问内存
- 变址寻址:在间接寻址之时改变寄存器的数值 




## PUSH 指令


```
pushl %eax
```

将eax里的内容推到栈顶内存中

相当于
```
subl $4,%esp
movl %eax,(%esp)
```

- 堆栈是向下增长的
- esp 是堆栈寄存器,存放指向栈顶的内存地址

## POP 指令

```
popl %eax
```

相当于
```
movl (%esp),%eax
addl $4,%esp
```

将堆栈的栈顶的数据弹出(出栈),放到eax中

## CALL 和 RET

```
call 0x10A00C
```

相当于
```
pushl %eip
movel $0x10A00C,%eip
```

---

```
ret
```
相当于
```
popl %eip
```

注意:eip不能被程序员直接使用,必须使用相关指令间接使用(例如使用 call指令).



