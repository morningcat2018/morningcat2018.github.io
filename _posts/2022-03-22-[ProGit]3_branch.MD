---
layout:     post
title:      Pro Git笔记 3.分支
subtitle:   分支
date:       2022-03-22
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - ProGit
---

[Pro Git 文档](https://git-scm.com/book/zh/v2)

## 分支简介

假设现在有一个工作目录，里面包含了三个将要被暂存和提交的文件

> git add README test.rb LICENSE

> git commit -m 'commit of my project'

当使用 `git commit` 进行提交操作时，Git 会先计算每一个子目录（本例中只有项目根目录）的校验和， 然后在 Git 仓库中这些校验和保存为`树对象`。随后，Git 便会创建一个`提交对象`， 它除了包含上面提到的那些信息外，还包含指向这个树对象（项目根目录）的指针。

现在，Git 仓库中有五个对象：三个 `blob 对象`（保存着文件快照）、一个 `树 对象` （记录着目录结构和 blob 对象索引）以及一个 `提交 对象`（包含着指向前述树对象的指针和所有提交信息）。

![首次提交对象及其树结构](/png/git/git-branch-001.png)

做些修改后再次提交，那么这次产生的提交对象会包含一个指向上次提交对象（父对象）的指针。

![提交对象及其父对象](/png/git/git-branch-002.png)

`Git 的分支，其实本质上仅仅是指向提交对象的可变指针`。

---

Git 的 master 分支并不是一个特殊分支。 它就跟其它分支完全没有区别。 之所以几乎每一个仓库都有 master 分支，是因为 git init 命令默认创建它，并且大多数人都懒得去改动它。

![图表 11. 分支及其提交历史](https://img-blog.csdnimg.cn/452eae7ad12a43efb1eccb80392f01b4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbW9ybmluZ2NhdDIwMTg=,size_20,color_FFFFFF,t_70,g_se,x_16)


## 分支创建

创建一个 testing 分支

> git branch testing

![两个指向相同提交历史的分支](https://img-blog.csdnimg.cn/4f59cb1b216d46f4aff0694d496b09d6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbW9ybmluZ2NhdDIwMTg=,size_20,color_FFFFFF,t_70,g_se,x_16)

那么，Git 又是怎么知道当前在哪一个分支上呢？ 也很简单，它有一个名为 HEAD 的特殊指针。

在本例中，你仍然在 master 分支上。 因为 `git branch 命令`仅仅 创建 一个新分支，并不会自动切换到新分支中去。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20d23c7e40144b4cac541a46eaafefc4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbW9ybmluZ2NhdDIwMTg=,size_20,color_FFFFFF,t_70,g_se,x_16)

## 分支切换

需要使用 `git checkout 命令`

> git checkout testing

这样 `HEAD` 就指向 testing 分支了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/b74b488ecb67460f85d7bbc2d0304c96.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbW9ybmluZ2NhdDIwMTg=,size_20,color_FFFFFF,t_70,g_se,x_16)

做一些修改后提交

> vim test.rb
> git commit -a -m 'made a change'

![在这里插入图片描述](https://img-blog.csdnimg.cn/b2e4044eef874f68a0cb2d51bef2c648.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbW9ybmluZ2NhdDIwMTg=,size_20,color_FFFFFF,t_70,g_se,x_16)

---

切换回 master 分支

> git checkout master

![在这里插入图片描述](https://img-blog.csdnimg.cn/f03f7511e80d41e88efb62aafe57093d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbW9ybmluZ2NhdDIwMTg=,size_20,color_FFFFFF,t_70,g_se,x_16)

这条命令做了两件事。 一是`使 HEAD 指回 master 分支`，二是`将工作目录恢复成 master 分支所指向的快照内容`。

我们不妨再稍微做些修改并提交：

> vim test.rb
> git commit -a -m 'made other changes'

现在，这个项目的提交历史已经产生了分叉。 

![在这里插入图片描述](https://img-blog.csdnimg.cn/2511b03e91624437821c84e7eff760a2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbW9ybmluZ2NhdDIwMTg=,size_20,color_FFFFFF,t_70,g_se,x_16)

#### 查看分叉历史

> git log --oneline --decorate --graph --all


#### 创建新分支的同时切换过去

> git checkout -b [newbranchname]

## 分支的合并

> git checkout master

将 issue33 分支合并到 master 分支上

> git merge issue33


#### 遇到冲突时的分支合并

有时候合并操作不会如此顺利。 如果你在两个不同的分支中，对`同一个文件的同一个部分进行了不同的修改`，Git 就没法`干净的合并`它们。

> git merge iss53

此时 Git 做了合并，但是没有自动地创建一个新的合并提交。 Git 会`暂停下来`，等待你去解决合并产生的冲突。你可以在合并冲突后的任意时刻使用 `git status 命令`来查看那些因包含合并冲突而处于`未合并（unmerged）状态的文件`

任何因包含合并冲突而有`待解决的文件`，都会以`未合并状态`标识出来。 Git 会在有冲突的文件中加入`标准的冲突解决标记`，这样你可以打开这些包含冲突的文件然后手动解决冲突。 出现冲突的文件会包含一些特殊区段，看起来像下面这个样子：

```
<<<<<<< HEAD:index.html
<div id="footer">contact : email.xxx@github.com</div>
=======
<div id="footer">
please contact us at yyy@github.com
</div>
>>>>>>> iss53:index.html
```

这表示 HEAD 所指示的版本（当前分支）在这个区段的上半部分（======= 的上半部分），而 iss53 分支所指示的版本在 ======= 的下半部分。 为了解决冲突，你必须选择使用由 ======= 分割的两部分中的一个，或者你也可以自行合并这些内容

在你解决了所有文件里的冲突之后，对每个文件使用 `git add 命令`来将其标记为冲突已解决。 一旦暂存这些原本有冲突的文件，Git 就会将它们标记为冲突已解决。

---

如果你想使用图形化工具来解决冲突，你可以运行 `git mergetool`，该命令会为你启动一个合适的可视化合并工具，并带领你一步一步解决这些冲突

## 分支管理

当前所有分支的列表

> git branch

注意` * 字符`标识的当前 HEAD 指针所指向的分支

---

如果需要查看每一个分支的最后一次提交

> git branch -v

`--merged` 与 `--no-merged` 这两个有用的选项可以过滤这个列表中已经合并或尚未合并到当前分支的分支。如果要查看哪些分支已经合并到当前分支，可以运行 `git branch --merged`

> git branch --merged

#### 删除分支

> git branch -d iss01

强制删除

> git branch -D testing

## 分支开发工作流

#### 长期分支

在整个项目开发周期的不同阶段，你可以同时拥有多个开放的分支；你可以定期地把某些主题分支合并入其他分支中

#### 主题分支

主题分支是一种短期分支，它被用来实现单一特性或其相关工作。

## 远程分支

远程引用是对远程仓库的引用（指针），包括分支、标签等等。

显式地获得远程引用的完整列表  `git ls-remote  <remote>`

> git ls-remote origin

获得远程分支的更多信息 `git remote show <remote>`

> git remote show origin

与给定的远程仓库同步数据 `git fetch <remote>`

> git fetch origin

推送到有写入权限的远程仓库上，运行 `git push <remote> <branch>`

> git push origin serverfix

如果并不想让远程仓库上的分支叫做 serverfix，可以运行 `git push origin serverfix:awesomebranch` 来将`本地的 serverfix 分支`推送到`远程仓库上的 awesomebranch 分支`。

将远程分支内容合并到当前分支

> git merge origin/serverfix

拉取远程分支到本地

> git checkout -b serverfix origin/serverfix

#### 跟踪分支

??

#### 拉取

`git pull 命令`：它的含义是一个 `git fetch` 紧接着一个`git merge 命令`。

git pull 会查找当前分支所跟踪的服务器与分支， 从服务器上抓取数据然后尝试合并入那个远程分支。

#### 删除远程分支

> git push origin --delete serverfix

## 变基

??

