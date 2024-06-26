---
layout:     post
title:      Pro Git笔记 1.起步
subtitle:   起步
date:       2022-03-22
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - ProGit
---


[Pro Git 文档](https://git-scm.com/book/zh/v2)

## 关于版本控制

1. 集中化的版本控制系统

有一个单一的集中管理的`服务器`，保存`所有文件的修订版本`，
而协同工作的人们都通过`客户端`连到这台服务器，取出`最新的文件`或者提交更新。

如：CVS，SVN 等等

2. 分布式版本控制系统

客户端并不只提取最新版本的文件快照， 而是把代码仓库完整地镜像下来，包括完整的`历史记录`。
每一次的克隆操作，实际上都是一次对代码仓库的`完整备份`。

如：Git、Mercurial、Bazaar 以及 Darcs 等

## Git 是什么？

1. 直接记录快照，而非差异比较

Git 更像是把数据看作是对小型文件系统的一系列`快照`：
在 Git中，每当你提交更新或保存项目状态时，它基本上就会对当时的全部文件创建一个快照并保存这个快照的索引。
为了效率，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。

![在这里插入图片描述](/png/git/git001-1.png)

2. 近乎所有操作都是本地执行

在 Git 中的绝大多数操作都只需要访问本地文件和资源，一般不需要来自网络上其它计算机的信息。
因为你在本地磁盘上就有项目的完整历史，所以大部分操作看起来瞬间完成。

3. Git 保证完整性

Git 用以计算校验和的机制叫做 `SHA-1 散列`（hash，哈希）。 这是一个由 40 个十六进制字符（0-9 和 a-f）组成的字符串，基于 Git 中文件的内容或目录结构计算出来。
> 24b9da6552252987aa493b52f8696cd6d3b00373

实际上，Git 数据库中保存的信息都是以文件内容的哈希值来索引，而不是文件名。

4. Git 一般只添加数据

也就是说 Git 几乎不会执行任何可能导致文件不可恢复的操作。 

5. 三种状态

Git 有三种状态，你的文件可能处于其中之一： 
- `已修改` 表示修改了文件，但还没保存到数据库中。
- `已暂存` 表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。
- `已提交` 表示数据已经安全地保存在本地数据库中。

这会让我们的 Git 项目拥有三个阶段：工作区、暂存区以及 Git 目录

![在这里插入图片描述](/png/git/git001-2.png)

## 安装 Git

安装部分省略...

测试安装是否成功 
```
> git --version
git version 2.32.0 (Apple Git-132)
```

## 初次运行 Git 前的配置

Git 自带一个 git config 的工具来帮助设置控制 Git 外观和行为的配置变量。 这些变量存储在三个不同的位置：

- `/etc/gitconfig 文件`: 包含系统上每一个用户及他们仓库的`通用配置`。 如果在执行 git config 时带上 `--system` 选项，那么它就会读写该文件中的配置变量。 
- `~/.gitconfig 或 ~/.config/git/config 文件`：只针对当前用户。 你可以传递 `--global` 选项让 Git 读写此文件，这会对你系统上 所有 的仓库生效。
- 当前使用仓库的 Git 目录中的 config 文件（即 `.git/config`）：针对该仓库。 你可以传递 `--local` 选项让 Git 强制读写此文件

每一个级别会覆盖上一级别的配置

---

1. 查看所有的配置

>  git config --list --show-origin

>  git config --list

2. 用户信息

安装完 Git 之后，要做的第一件事就是设置你的用户名和邮件地址。 
这一点很重要，因为每一个 Git 提交都会使用这些信息，它们会写入到你的每一次提交中，不可更改

> git config --global user.name "Your Name"
> git config --global user.email my-email@email.com

3. 文本编辑器

当 Git 需要你输入信息时会调用它

4. 检查配置信息  git config <key>

>  git config user.name


## 获取帮助

1. git help <verb>

>  git help config

>  git help add

2.  git <verb> --help

> git add --help

> git add -h





