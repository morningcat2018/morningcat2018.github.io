---
layout:     post
title:      jenkins笔记
subtitle:   历史笔记梳理
date:       2024-05-20
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_652.jpg
catalog: true
tags:
    - jenkins
---


#### 下载

http://mirrors.jenkins.io/war/latest/jenkins.war

#### 启动

> java -jar jenkins.war

```
Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:
121dcc95950f48b482963148b0997bda
This may also be found at: C:\Users\xxx\.jenkins\secrets\initialAdminPassword
```

在浏览器上打开：http://127.0.0.1:8080

会提示：为了确保管理员安全地安装jenkins，密码已写入到日志中（不知道在哪里？）该文件在服务器上：C:\Users\xxx\.jenkins\secrets\initialAdminPassword

#### 附录

- java -jar jenkins.war启动，默认使用的是内置的jetty容器; 也可以将war放到tomcat容器中启动
- Linux环境下后台运行 : nohup java -jar jenkins.war >temp.txt &
- 若重新使用 java -jar jenkins.war 启动，首先会查找是否存在曾经启动过后保存的配置

#### 插件

1 插件管理

系统管理->插件管理
在可选插件中可以自主安装插件


2 管理用户

系统管理->管理用户->新建用户

3 安全配置

系统管理->全局安全配置

授权策略 选择安全矩阵
然后添加现有的用户，赋予权限
（如果不进行安全配置，默认权限就是管理员权限，拥有所有权限）


4 管理服务器节点

系统管理->节点管理
新建节点
输入名称，选择固定节点
设置远程工作目录
启动方式选择：Launch slave agents via SSH
主机填入ip
点击Add填入用户名、密码
在Credentials选中刚刚填入的用户名和密码

测试节点
在节点列表中，点击节点名称；启动代理

#### 脚本

​```bash
#!/bin/sh
# export PROJ_PATH=项目路径
# export TOMCAT_PATH=tomcat路径

killTomcat()
{
    pid=`ps -ef | grep tomcat | grep java|awk '{print $2}'`
    echo "tomcat Id list :$pid"
    if ["$pid" = ""]
    then
        echo "no tomcat pid alive"
    else
        kill -9 $pid
    fi
}
```









​