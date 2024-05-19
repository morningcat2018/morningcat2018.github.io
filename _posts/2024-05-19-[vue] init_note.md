---
layout:     post
title:      Vue 入门笔记
subtitle:   vue2
date:       2024-05-19
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_652.jpg
catalog: true
tags:
    - Vue
---

## 前端知识体系

前端三要素

- HTML
- CSS 标记语言
- JS

#### CSS预处理器

用一种专门的编程语言进行样式设计 再通过编译器转换成CSS文件

- SASS : 基于 Ruby 通过服务端处理 功能强大 解析效率高
- LESS : 基于 NodeJs 通过客户端处理 上手简单

#### Native 原生 JS 开发

ECMAScript 标准

- ES3 
- ES4 未发布
- ES5 浏览器广泛支持
- ES6 当前主流版本
- ES7
- ES8

#### TypeScript

- 微软开源的编程语言
- JavaScript 的一个超集
- 需要编译成JS之后运行

#### JavaScript 框架

- jQuery
    - 简化DOM操作
- Angular
    - 采用 TypeScript 语法开发
    - 对后端友好,对前端来说不友好
- React
    - 高性能
    - 虚拟DOM 在内存中模拟DOM操作 提升渲染效率
    - 需要学习 JSX 语言
- Vue
    - 渐进式
    - `只负责视图层`
- Axios
    - 前端`通信`框架
    - 与 jQuery 提供的 AJAX 通信功能类似

#### UI 框架

- Ant-Design 基于 React 的 UI 框架
- ElementUI 基于 Vue 的 UI 框架
- BootStrap 前端开发的开源工具包

#### JavaScript 构建工具

- Babel : JS编译工具 比如用于编译 TypeScript
- webpack : 模块打包器 打包、压缩、合并、按序加载


## NodeJs

全栈时代

> node -v

> npm -v

## MVVM 模式 (Model-View-ViewModel)

- 一种软件架构设计模式
- 一种事件驱动编程方式
- 源于经典的MCV模式(Model-View-Controller)
- `核心是 ViewModel 层` (Vue 就是 ViewModel 层框架)
    - 负责转换 Model 中的数据对象来让数据变得更容易管理和使用
    - 向上与视图层进行双向数据绑定
    - 向下与Model层通过接口请求进行数据交互


## Vue

- https://cn.vuejs.org/
- https://v2.cn.vuejs.org/v2/guide/

#### 概述

- 用于构建用户界面的渐进式框架
- `只关注视图层`
- 易于与现代化的工具链结合使用
- MVVM 模式的实现者: Vue.js 就是 MVVM 中的 ViewModel 层的实现者

#### 核心要素

- 数据驱动
- 组件化

#### Vue 应用程序

创建一个 .html 文件

```html
<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1.0">
  <script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
  <title>myvuedemo</title>
</head>

<body>
  <div id="app">
    {{ message }}
  </div>

  <script>
    var app = new Vue({
      el: '#app',
      data: {
        message: 'Hello Vue!'
      }
    })
  </script>
</body>

</html>
```

可安装官方教程学习以及练习

#### Axios 框架

https://github.com/axios/axios

- 异步通信框架
- 可以用于浏览器与 NodeJs 之间的异步通信
- 支持 Promise API
- 支持防御 XSRF


#### vue-cli 脚手架

> npm install vue-cli -g

install path /usr/local/lib/node_modules/

> vue init webpack myvuedemo

myvuedemo Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```

#### webpack

> npm install webpack -g

> npm install webpack-cli -g

> webpack --watch


#### vue-router

- 路由管理器
- 与Vue.js核心深度集成

#### 整合 ElementUI

> vue init webpack vue-elementui


