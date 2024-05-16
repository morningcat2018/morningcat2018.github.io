---
layout:     post
title:      Python 下的虚拟环境的使用
subtitle:   
date:       2019-12-13
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - python
---


## 虚拟环境分类

- pyenv: 多个解释器，Python不同版本的隔离
- pyvenv: 一个解释器，项目隔离(包隔离) python3.4开始自带默认(常用)
- virtualenv: 一个解释器，项目隔离(包隔离)，第三方pypi。支持 2.6~3.5 版本


#### pyvenv 使用

```
pip install pyvenv          # 安装，python3.4默认安装
mkdir python_env
cd python_env

pyvenv env_**               # 新建env_**虚拟环境(pyCharm下默认为 venv)
source env_**/bin/activate  # 使用虚拟环境
deactivate                  # 退出虚拟环境
记录虚拟环境的软件包依赖关系：
pip freeze > [文件名]
```

> 尽管Python 3.7.3 的 bin 目录下依然有 pyvenv 的脚本，但是在打印的help信息的第一行明确警告说：这个脚本是过时的，推荐使用 `python3.7 -m venv` 命令

---

#### 虚拟环境 virtualenv

```
pip install virtualenv

virtualenv [xxx]        # 创建虚拟环境,创建成功后会生成一个文件夹
[xxx]\Scripts\activate  # 激活并使用虚拟环境

Python3.3以上的版本通过venv模块原生支持虚拟环境，可以代替Python之前的virtualenv。该venv模块提供了创建轻量级“虚拟环境”，提供与系统Python的隔离支持。每一个虚拟环境都有其自己的Python二进制（允许有不同的Python版本创作环境），并且可以拥有自己独立的一套Python包。Python3.3中使用”venv”命令创建的环境不包含”pip”，你需要进行手动安装。在Python3.4中改进了这一个缺陷。
```
---

查看依赖库的安装位置

> python -m site 

查看pip安装过的库

> pip freeze

---

## pip

```
pip --version # 显示版本路径
pip --help # 帮助
pip install --upgrade somePackage # 升级somePackage
easy_install --upgrade somePackage # 如果升级出现问题，可以使用
pip uninstall somePackage # 卸载somePackage模块
pip search somePackage # 搜索somePackage模块
pip show -f somePackage # 显示指定包的详细信息
```

---

```bash
cd /usr/local/bin
ln -snf ../Cellar/python/3.7.3/bin/pip3 pip
ln -snf ../Cellar/python/3.7.3/bin/python3 python
```

---

参考资料：

https://blog.csdn.net/wangye1989_0226/article/details/84862788


## 2024 重新梳理笔记

- 当前环境python3.12

---

开启虚拟环境

> mkdir my-project-path && cd my-project-path

> python -m venv env

在Linux环境或MAC环境

> source env/bin/activate

在Windows环境

> env\Scripts\activate.bat

## pip 安装使用国内镜像


临时使用国内镜像源

> pip install xxx -i https://pypi.tuna.tsinghua.edu.cn/simple

永久使用国内镜像源

> pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

> pip config list



除了清华大学的镜像源，还有其他一些常用的镜像源包括：

- 阿里云：https://mirrors.aliyun.com/pypi/simple/
- 中国科技大学：https://pypi.mirrors.ustc.edu.cn/simple/
- 豆瓣(douban)：http://pypi.douban.com/simple/
- 华为云：https://mirrors.huaweicloud.com/repository/pypi/simple
