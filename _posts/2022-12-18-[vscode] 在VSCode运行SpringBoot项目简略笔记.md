---
layout:     post
title:      在VSCode运行SpringBoot项目简略笔记
subtitle:   
date:       2022-12-18
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - vscode
---


## 背景

1. 已安装JDK，并配置环境变量
2. 已安装Maven，并配置环境变量

## VSCode

1. 下载并安装VSCode

[VSCode 网址](https://code.visualstudio.com/)

2. 配置插件

- Extension Pack for Java
- Spring Boot Extension Pack

3. 打开工程

文件 -> 打开文件夹

4. 配置 VM options

点击左下角的设置按钮，点击设置选项；

搜索 launch ，点击 在settings.json中编辑

添加configurations配置，如下所示：

```java
{
    "launch": {       
        "configurations": [
            {
                "name": "Java: Current File",
                "type": "java",
                "request": "launch",
                "mainClass": "com.morning.Main", // main class 类路径
                "vmArgs": "-Dfile.encoding=UTF-8 -Dserver.port=9203 " // 需要设置的参数
            }
        ]
    }
}
```

5. 运行

在 Main.java(SpringBoot的启动类，各个工程命名不同) 中右键，点击 Run Java

## 参考资料

- https://juejin.cn/post/7132284109186924574#heading-0
- https://blog.csdn.net/m0_51426055/article/details/126391894