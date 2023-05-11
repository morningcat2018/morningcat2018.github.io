

## docsify 简介

docsify 是一种`文档网站生成器`；

docsify 可以快速帮你生成文档网站。不同于 GitBook、Hexo 的地方是它不会生成静态的 .html 文件，所有转换工作都是在运行时。

官网：[https://docsify.js.org/#/zh-cn/](https://docsify.js.org/#/zh-cn/)

动态文档生成工具有很多，如：Docsify、 VuePress、Docute 、Hexo

## 安装部署

> 前提：安装 node 和 npm 

执行命令进行安装

> npm i docsify-cli -g

## 初始化项目

> docsify init ./my-pro

> cd my-pro

```
> ll -a
-rw-r--r--  1 morningcat  staff     0B 10 26  1985 .nojekyll
-rw-r--r--  1 morningcat  staff    34B 10 26  1985 README.md
-rw-r--r--  1 morningcat  staff   604B  1 18 23:08 index.html
```

- index.html 入口文件
- README.md 会做为主页内容渲染
- .nojekyll 用于阻止 GitHub Pages 忽略掉下划线开头的文件

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="description" content="Description">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0">
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify@4/lib/themes/vue.css">
</head>

<body>
  <div id="app"></div>
  <script>
    window.$docsify = {
      name: '',
      repo: ''
    }
  </script>
  <!-- Docsify v4 -->
  <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
</body>

</html>
```

## 本地预览

通过运行 `docsify serve` 启动一个本地服务器，可以方便地实时预览效果。默认访问地址 [http://localhost:3000](http://localhost:3000)

> docsify serve

---

## 定制侧边栏

1. 在 `index.html` 中添加相关内容

```html
<script>
  window.$docsify = {
    loadSidebar: true
  }
</script>
<script src="//cdn.jsdelivr.net/npm/docsify/lib/docsify.min.js"></script>
```

2. 创建 `_sidebar.md` 文件

```
* [首页](/README)    <!-- 对应 README.md -->
* [目录1](001) <!-- 对应 001.md -->
* [目录2](002 "这里是提示，可有可无") <!-- 对应 002.md -->
```

## 显示目录

在 `index.html` 中添加 subMaxLevel: 2

```html
<script>
  window.$docsify = {
    loadSidebar: true,
    subMaxLevel: 2,
  }
</script>
<script src="//cdn.jsdelivr.net/npm/docsify/lib/docsify.min.js"></script>
```

当设置了 subMaxLevel 时，默认情况下每个标题都会自动添加到目录中。如果你想忽略特定的标题，可以给它添加 <!-- {docsify-ignore} --> 

```md
## 222

Hello 世界

## 333 <!-- {docsify-ignore} --> 

该标题不会出现在侧边栏的目录中
```

## 自定义导航栏

1. 在 `index.html` 中添加相关内容  `<nav></nav>` 和 `loadNavbar: true`

```html
<body>
  <nav>
  </nav>

  <div id="app"></div>

  <script>
    window.$docsify = {
        loadNavbar: true
    }
  </script>
</body>

<script src="//cdn.jsdelivr.net/npm/docsify/lib/docsify.min.js"></script>
```

2. 创建 `_navbar.md` 文件

```
* 入门
  * [快速开始](zh-cn/quickstart.md)
  * [多页文档](zh-cn/more-pages.md)
  * [定制导航栏](zh-cn/custom-navbar.md)
  * [封面](zh-cn/cover.md)


* 配置
  * [配置项](zh-cn/configuration.md)
  * [主题](zh-cn/themes.md)
  * [使用插件](zh-cn/plugins.md)
  * [Markdown 配置](zh-cn/markdown.md)
  * [代码高亮](zh-cn/language-highlight.md)

* 链接到我
  * [Github地址](https://github.com/morningcat2018)
  * [CSDN](https://blog.csdn.net/u013837825)
```

## emoji 插件

在 `index.html` 中添加 

```html
<script src="//cdn.jsdelivr.net/npm/docsify/lib/plugins/emoji.min.js"></script>
```

## 封面

1. 在 `index.html` 中添加 

```html
<script>
  window.$docsify = {
    coverpage: true
  }
</script>
<script src="//cdn.jsdelivr.net/npm/docsify/lib/docsify.min.js"></script>
```

2. 创建 `_coverpage.md` 文件

```
<!-- _coverpage.md -->

![logo](_media/icon.svg)

# docsify <small>3.5</small>

> 一个神奇的文档网站生成器。

- 简单、轻便 (压缩后 ~21kB)
- 无需生成 html 文件
- 众多主题

[GitHub](https://github.com/docsifyjs/docsify/)
[Get Started](/README)
```

## 更多内容

参考官网 [https://docsify.js.org/#/zh-cn/](https://docsify.js.org/#/zh-cn/)

## 参考资料

- [官网](https://docsify.js.org/#/zh-cn/)
- https://www.cnblogs.com/Can-daydayup/p/15413267.html