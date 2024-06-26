---
layout:     post
title:      Pro Git笔记 2.基础
subtitle:   基础
date:       2022-03-22
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - ProGit
---

[Pro Git 文档](https://git-scm.com/book/zh/v2)

## 创建 Git 仓库

1. 在已存在目录中初始化仓库

> cd my-workspace

> mkdir my-prpject-a

> cd my-prpject-a

> git init

`git init` 该命令将创建一个名为 .git 的子目录，这个子目录含有你初始化的 Git 仓库中所有的必须文件，这些文件是 Git 仓库的骨干

2. 克隆现有的仓库

> git clone https://github.com/your-name/your-project

> cd your-project

---

> git clone https://github.com/your-name/your-project my-project-a

> cd my-project-a

## 记录每次更新到仓库【重点】

1. 检查当前文件状态

>  git status 

查看哪些文件处于什么状态

---

案例：

> echo 'My Project' > README

> git status

```
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	README

nothing added to commit but untracked files present (use "git add" to track)
```

2. 跟踪新文件 

> git add [file]

---

案例：

> git add README

> git status

```
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   README
```

只要在 `Changes to be committed` 这行下面的，就说明是已暂存状态。 

如果此时提交，那么该文件在你`运行 git add 时的版本`将被留存在后续的历史记录中。
git add 命令使用文件或`目录`的路径作为参数；如果参数是目录的路径，该命令将递归地跟踪该目录下的所有文件。

3. 暂存已修改的文件

修改一下 README 文件

> git status

```
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   README

```

文件同时出现在暂存区和非暂存区；

`实际上 Git 只不过暂存了你运行 git add 命令时的版本`。 如果你现在提交， README 的版本是你最后一次运行 git add 命令时的那个版本，而不是你运行 git commit 时的版本。

所以，运行了 git add 之后又作了修订的文件，需要重新运行 git add 把最新版本重新暂存起来

---

 git add 命令 是个多功能命令：
 - 可以用它开始跟踪新文件
 - 把已跟踪的文件放到暂存区
 - 用于合并时把有冲突的文件标记为已解决状态
 - 将这个命令理解为“`精确地将内容添加到下一次提交中`”而不是“将一个文件添加到项目中”要更加合适


4. 状态简览

> git status -s 

> git status --short

将得到一种格式更为`紧凑`的输出

- ?? 标记 : 新添加的未跟踪文件
- A 标记 : 新添加到暂存区中的文件
- M 标记 : 修改过的文件

5. 忽略文件

一般我们总会有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。 通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等

创建一个名为 `.gitignore` 的文件，列出要忽略的文件的模式

```
.DS_Store
target/
.idea/*
*.iml
```

---

文件 `.gitignore` 的格式规范如下：
- 所有空行或者以 # 开头的行都会被 Git 忽略。
- 可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中。
- 匹配模式可以以（/）开头防止递归。
- 匹配模式可以以（/）结尾指定目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（!）取反。

所谓的 `glob 模式`是指 shell 所使用的简化了的正则表达式。 
- 星号（*）匹配零个或多个任意字符；
- [abc] 匹配任何一个列在方括号中的字符 （这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）； 
- 问号（?）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符， 表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）。 
- 使用两个星号（`**`）表示匹配任意中间目录，比如 a/**/z 可以匹配 a/z 、 a/b/z 或 a/b/c/z 等。

6. 查看已暂存和未暂存的修改

> `git diff`

此命令比较的是工作目录中当前文件和暂存区域快照之间的差异。 也就是修改之后还没有暂存起来的变化内容。

> git diff --staged

查看已暂存的将要添加到下次提交里的内容

> git diff --cached

查看已经暂存起来的变化（ --staged 和 --cached 是同义词）

---

可以使用 git difftool 命令来调用 emerge 或 vimdiff 等软件（包括商业软件）输出 diff 的分析结果。

7. 提交更新

> git commit -m "git log"

8. 跳过使用暂存区域

> git commit -a -m "git log"

git add 和 git commit 的组合命令

这很方便，但是要小心，有时这个选项会将不需要的文件添加到提交中


9. 移除文件

>  git rm [file>

如果要删除之前修改过或已经放到暂存区的文件，则必须使用强制删除选项 `-f`（译注：即 force 的首字母）

---

另外一种情况是，我们想把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。 
换句话说，你想让文件保留在磁盘，但是并不想让 Git 继续跟踪。 当你忘记添加 .gitignore 文件，不小心把一个很多的日志文件添加到暂存区时，这一做法尤其有用。 
为达到这一目的，使用 `--cached` 选项：

>  git rm --cached [file>

## 查看提交历史

> git log 

不传入任何参数的默认情况下，git log 会按时间先后顺序列出所有的提交，最近的更新排在最上面。
这个命令会列出每个提交的 SHA-1 校验和、作者的名字和电子邮件地址、提交时间以及提交说明

---

> git log -p

显示每次提交所引入的差异（按 补丁 的格式输出）；
也可以限制显示的日志条目数量，例如使用 `-2` 选项来只显示`最近的两次提交` 
> git log -p -2

---

> git log --stat

能看到每次提交的简略统计信息

`--stat 选项`在每次提交的下面列出所有被修改过的文件、有多少文件被修改了以及被修改过
的文件的哪些行被移除或是添加了。 在每次提交的最后还有一个总结。

---

> git log --pretty=oneline

可以使用不同于默认格式的方式展示提交历史。 这个选项有一些内建的子选项供你使用。 比如 `oneline` 会将每个提交放在一行显示，在浏览大量的提交时非常有用。 另外还有 `short`，`full` 和 `fuller` 选项，

最有意思的是 format ，可以定制记录的显示格式。 这样的输出对后期提取分析格外有用——因为你知道输出的格式不会随着 Git 的更新而发生改变：

> git log --pretty=format:"%h - %an, %ar : %s"

[git log --pretty=format 常用的选项](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2#pretty_format)

---

限制输出长度：类似 --since 和 --until 这种按照时间作限制的选项很有用。

> git log --since=2.weeks

---

还可以过滤出匹配指定条件的提交。 用 `--author` 选项显示指定作者的提交，用 `--grep` 选项搜索提交说明中的关键字。

[git log 的常用选项 ](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2#log_options)

## 撤消操作

> git commit --amend

---

2. 取消暂存的文件

> git reset HEAD [file>

## 远程仓库的使用

1. 查看远程仓库

> git remote -v

2. 添加远程仓库

>  git remote add [remote> https://github.com/xxx/xxx

没有 [remote> ，默认为 origin

3. 从远程仓库中抓取与拉取

> git fetch [remote>

4. 推送到远程仓库

> git push [remote> master

5. 查看某个远程仓库

>  git remote show [remote>

如果你是 Git 的重度使用者，那么还可以通过 git remote show 看到更多的信息。

6. 远程仓库的重命名与移除

> git remote rename [remote1> [remote2>

> git remote remove [remote>


## 打标签

1. 列出标签

> git tag

> git tag -l "v1.8.5*"

2. 创建标签

Git 支持两种标签：轻量标签（lightweight）与附注标签（annotated）

轻量标签很像一个不会改变的分支——它只是某个特定提交的引用

附注标签是存储在 Git 数据库中的一个完整对象

---

创建附注标签

> git tag -a v1.4 -m "my version 1.4"

通过使用 git show 命令可以看到标签信息和与之对应的提交信息

> git show v1.4

---

创建轻量标签

> git tag v1.5

> git show v1.5

3. 后期打标签

> git log --pretty=oneline

> git tag -a v1.2 8a5cbc430f1a9c3d00faaeffd07798508422908a

4. 共享标签

默认情况下，git push 命令并不会传送标签到远程仓库服务器上。 在创建完标签后你必须显式地推送标签到共享服务器上。 这个过程就像共享远程分支一样——你可以运行 
> git push origin [tagname>

> git push origin --tags

5. 删除标签

> git tag -d v1.4

注意上述命令并不会从任何远程仓库中移除这个标签，你必须用 `git push [remote>:refs/tags/[tagname>` 来更新你的远程仓库

第一种变体是 git push [remote> :refs/tags/[tagname> ：
> git push origin :refs/tags/v1.4

第二种更直观的删除远程标签的方式是：
>  git push origin --delete v1.4

6. 检出标签

> git checkout v1.5


## Git 别名

Git 并不会在你输入部分命令时自动推断出你想要的命令。 如果不想每次都输入完整的 Git 命令，可以通过 git config 命令来轻松地为每一个命令设置一个别名。

>  git config --global alias.co checkout

> git config --global alias.br branch

> git config --global alias.ci commit

> git config --global alias.st status

意味着，当要输入 `git commit` 时，只需要输入 `git ci` 就可以了

---

> git config --global alias.unstage 'reset HEAD --'

> git config --global alias.last 'log -1 HEAD'

> git config --global alias.visual '!gitk'