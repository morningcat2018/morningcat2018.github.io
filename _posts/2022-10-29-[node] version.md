---
layout:     post
title:      node版本
subtitle:   
date:       2022-10-29
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_62a.jpg
catalog: true
tags:
    - node
---


# 查看当前node版本
$ node -v

# 清除npm缓存
$ npm cache clean -f

# 全局安装n
$ npm install -g n

# 升级到最新稳定版
$ n stable

# 升级到最新版
$ n latest

# 升级到定制版
$ n v14.6.0

# 切换使用版本
$ n 13.10.0 (ENTER)

# 删除制定版本
$ n rm 13.10.0

# 用制定的版本执行脚本
$ n use 13.10.0 some.js

# 升级完成查看 node版本
$ node -v
