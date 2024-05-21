---
layout:     post
title:      docker笔记
subtitle:   历史笔记梳理
date:       2024-05-20
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_652.jpg
catalog: true
tags:
    - docker
---


## 基础命令

> docker version

查看本地镜像

> docker images

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
注意：这里的镜像id [IMAGE ID] 以后回经常用到;
```

拉取远程镜像到本地

> docker pull [image name]

如：
> docker pull hello-world

检查

> docker images

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              e38bc07ac18e        8 weeks ago         1.85kB
```

运行镜像

> docker run [image name]

如：
> docker run hello-world

```
Hello from Docker!
```

检查在运行的镜像

> docker ps

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

注意：
(1).这里的容器id [CONTAINER ID] 以后回经常用到;
(2).这里的hello-world只是一次性程序，不是持久性程序，所以这里docker ps 下没有结果
```

删除

> docker rmi [IMAGE ID]

> docker rm [CONTAINER ID]    删除

进入镜像内部目录

> docker exec -it [CONTAINER ID] /bin/bash

复制文件

> docker cp [file] [CONTAINER ID]://[路径]

## 安装nginx

获取镜像名称

- 阿里云docker镜像仓库 https://dev.aliyun.com/search.html
- 网易云docker镜像中心 https://c.163.com/hub#/m/home/

拉取镜像

> docker pull hub.c.163.com/library/nginx:latest

> docker images

```
REPOSITORY                                         TAG                 IMAGE ID            CREATED             SIZE
hello-world                                        latest              e38bc07ac18e        8 weeks ago         1.85kB
hub.c.163.com/library/tomcat                       latest              72d2be374029        10 months ago       292MB
hub.c.163.com/library/nginx                        latest              46102226f2fd        13 months ago       109MB
```

注意：这里的镜像名称为 hub.c.163.com/library/nginx


运行镜像

> docker run -d -p 8080:80 hub.c.163.com/library/nginx

或
> docker run -d -P hub.c.163.com/library/nginx

```
说明：
1. -d 指定后台运行
2. -p 指定映射端口  -P 随机映射端口
```

> docker ps

```
CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS              PORTS                    NAMES
b8d0f6c1f78c        hub.c.163.com/library/nginx    "nginx -g 'daemon of…"   5 seconds ago       Up 4 seconds        0.0.0.0:8080->80/tcp     vigorous_wiles
947be05b8eb6        hub.c.163.com/library/tomcat   "catalina.sh run"        11 minutes ago      Up 11 minutes       0.0.0.0:8888->8080/tcp   tender_kowalevski
```

检查端口

> netstat -na | grep 8080

进行镜像内

> docker exec -it [CONTAINER ID] [xx]

如：
> docker exec -it 109d4111fe79 /bin/sh

> docker exec -it fe0ae9dbc7ae /bin/bash

退出

> exit 

```
说明：
docker exec --help 获取帮助
```

拷贝文件到镜像内

> docker cp [file] [CONTAINER ID]://[路径]

如：
> docker cp index.html 109d4111fe79://usr/share/nginx/html

```
进入检查一下
docker exec -it 109d4111fe79 sh
# cd usr/share/nginx/html
# ls
50x.html  index.html
```

停止镜像

> docker stop [CONTAINER ID]

docker ps命令 获取[CONTAINER ID]

如：
> docker stop 0f37b66a5a0e

## 安装tomcat和MySQL

下载

docker pull hub.c.163.com/library/tomcat:latest
docker pull hub.c.163.com/library/mysql:latest

> docker images

```
REPOSITORY                                         TAG                 IMAGE ID            CREATED             SIZE
hello-world                                        latest              e38bc07ac18e        8 weeks ago         1.85kB
hub.c.163.com/library/tomcat                       latest              72d2be374029        10 months ago       292MB
hub.c.163.com/library/mysql                        latest              9e64176cd8a2        13 months ago       407MB
```

运行

> docker run -d -p 8888:8080 hub.c.163.com/library/tomcat


> docker ps

```
CONTAINER ID        IMAGE                                              COMMAND                  CREATED             STATUS              PORTS                                           NAMES
947be05b8eb6        hub.c.163.com/library/tomcat                       "catalina.sh run"        24 seconds ago      Up 23 seconds       0.0.0.0:8888->8080/tcp                          tender_kowalevski
```

停止

> docker stop 947be05b8eb6


运行mysql

> docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=mydatabase hub.c.163.com/library/mysql:latest

```
说明：
MYSQL_ROOT_PASSWORD 指定root密码
MYSQL_DATABASE 新建数据库
```


## 构建自己的docker镜像

这里做测试，使用jpress 下载war包 [地址1](https://gitee.com/fuhai/jpress/tree/alpha/wars)[地址2](https://github.com/JpressProjects/jpress)

> cp jpress-web-newest.war jpress.war

编写Dockerfile

> vim Dockerfile

```Dockerfile
from hub.c.163.com/library/tomcat
MAINTAINER mengzhang6 1213252320@qq.com
COPY jpress.war /usr/local/tomcat/webapps
```

构建

> docker build -t myweb:0.1 .

说明：最后的点 表示当前目录

> docker images

```
REPOSITORY                                         TAG                 IMAGE ID            CREATED             SIZE
myweb                                              0.1                 7a9d66d47a66        7 seconds ago       313MB
```

运行

> docker run -d -p 8888:8080 myweb:0.1

> docker ps

```
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS              PORTS                    NAMES
2e8a772a9f3a        myweb:0.1                            "catalina.sh run"        14 seconds ago      Up 12 seconds       0.0.0.0:8888->8080/tcp   thirsty_pare
c71cf6554cc0        hub.c.163.com/library/mysql:latest   "docker-entrypoint.s…"   14 minutes ago      Up 14 minutes       0.0.0.0:3306->3306/tcp   kind_stallman
b8d0f6c1f78c        hub.c.163.com/library/nginx          "nginx -g 'daemon of…"   About an hour ago   Up About an hour    0.0.0.0:8080->80/tcp     vigorous_wiles
```

重启

> docker restart [CONTAINER ID]

## 保存现有镜像更新后的镜像

我们在镜像内部做的任何更改，在镜像重启后都会复原; 
若我们希望保存我们所做的更改，可以保存更新后的版本为新的镜像

> docker commit -m 'message' [CONTAINER ID] [new image name]

实践：

> docker images | grep nginx

```
registry.cn-qingdao.aliyuncs.com/wecarepet/nginx   latest              f00bd095acb5        4 months ago        109MB
hub.c.163.com/library/nginx                        latest              46102226f2fd        13 months ago       109MB
```

> docker run -d -p 802:80 hub.c.163.com/library/nginx:latest

> docker ps | grep nginx

```
1eb03d91febb        hub.c.163.com/library/nginx:latest   "nginx -g 'daemon of…"   14 seconds ago      Up 11 seconds       0.0.0.0:802->80/tcp      youthful_varahamihira
```

> docker cp index.html 1eb03d91febb://usr/share/nginx/html

> docker commit -m 'message' 1eb03d91febb mycommit:0.1

```
sha256:8322dd2e2c10df2972de38fffbfcc4728eab0ccd60cddee7c230ea146c51d1d1
```

> docker images

```
REPOSITORY                                         TAG                 IMAGE ID            CREATED             SIZE
mycommit                                           0.1                 8322dd2e2c10        18 seconds ago      109MB
myweb                                              0.1                 7a9d66d47a66        21 minutes ago      313MB
hello-world                                        latest              e38bc07ac18e        8 weeks ago         1.85kB
```

> docker run -d -p 803:80 mycommit:0.1

> docker ps

```
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS              PORTS                    NAMES
cbbceb4aab7e        mycommit:0.1                         "nginx -g 'daemon of…"   6 seconds ago       Up 4 seconds        0.0.0.0:803->80/tcp      zealous_pasteur
1eb03d91febb        hub.c.163.com/library/nginx:latest   "nginx -g 'daemon of…"   5 minutes ago       Up 5 minutes        0.0.0.0:802->80/tcp      youthful_varahamihira
2e8a772a9f3a        myweb:0.1                            "catalina.sh run"        20 minutes ago      Up 20 minutes       0.0.0.0:8888->8080/tcp   thirsty_pare
c71cf6554cc0        hub.c.163.com/library/mysql:latest   "docker-entrypoint.s…"   34 minutes ago      Up 34 minutes       0.0.0.0:3306->3306/tcp   kind_stallman
```

## 推送镜像

查找镜像

> docker search whalesay

```
NAME                            DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
docker/whalesay                 An image for use in the Docker demo tutorial    631                                     
mendlik/docker-whalesay         Docker whalesay image from training material…   7                                       [OK]
sabs1117/whalesay               Whalesay with fortune phrases.                  1                                       
nikovirtala/whalesay            Tiny Go web service to print Moby Dock ASCII…   1                                       [OK]
```

拉取

> docker pull docker/whalesay

> docker images

```
REPOSITORY                                         TAG                 IMAGE ID            CREATED             SIZE
docker/whalesay                                    latest              6b362a9f73eb        3 years ago         247MB
```

运行

> docker run docker/whalesay cowsay 你好docker

生成自己的镜像

> docker tag docker/whalesay mengzhang6/whalesay

> docker login

推送到远程

> docker push mengzhang6/whalesay

## Dockerfile

编写Dockerfile

> vim Dockerfile

```Dockerfile
FROM alpine:latest
MAINTAINER morningcat
CMD echo 'hello docker!'
```

编译

> docker build -t hello-docker .

> docker images hello-docker

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-docker        latest              1c2ea804e9ae        15 seconds ago      4.15MB
```

运行

> docker run hello-docker

---


> mkdir image2 && cd image2

> vim Dockerfile

```Dockerfile
FROM hub.c.163.com/library/ubuntu
MAINTAINER morningcat
#RUN sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sorces.list
RUN apt-get update
RUN apt-get install -y nginx
COPY index.html /var/www/html
ENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"]
EXPOSE 80
```

> docker build -t mengzhang6/hello-nginx .

> docker run -d -p 80:80 mengzhang6/hello-nginx


Dockerfile语法

```
FROM    base image
RUN     执行命令
ADD     添加文件
COPY    拷贝文件
CMD     执行命令
EXPOSE  暴露端口
WORKDIR 指定路径
MAINTAINER 维护者
ENV     设定环境变量
ENTRYPOINT 容器入口
USER    指定用户
VOLUME  mount point
```

