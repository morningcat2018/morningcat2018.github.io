---
layout:     post
title:      Vue 上手单文件实践
subtitle:   vue2
date:       2024-05-30
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_652.jpg
catalog: true
tags:
    - Vue
---

- <https://v2.cn.vuejs.org/v2/guide/>
- <https://www.imooc.com/learn/980>

## 上手

下载[vue.js v2](https://v2.cn.vuejs.org/js/vue.js)

```md
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <title>myvuedemo</title>
  <script src="./vue.js"></script>
</head>

<body>
  <!-- 挂载点 -->
  <div id="app">
    {{ message }}
  </div>

  <script>
    var app = new Vue({
      el: '#app', // 指定挂载点
      data: {
        message: 'Hello Vue!'
      }
    })
  </script>
</body>

</html>
```

## 数据,事件与方法

```md
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <title>myvuedemo</title>
  <script src="./vue.js"></script>
</head>

<body>
  <div id="app">
    <h3>{{ message }}</h3>
    <h3 v-text="display_text"></h3>
    <h3 v-html="html_content"></h3>
    <div v-on:click="handle_event">测试点击事件</div>
    <div @click="handle_event2('morningcat')">测试点击事件2</div>
  </div>

  <script>
    var app = new Vue({
      el: '#app',
      data: {
        message: 'Hello Vue!',
        display_text: "你好啊",
        html_content: '<a href="https://www.bilibili.com/">bilibili</a>'
      },
      methods: {
        handle_event: function () {
          console.log('click')
          this.message = 'Hello World'
        },
        handle_event2: function (v) {
          this.message = 'Hello ' + v
        }
      }
    })
  </script>
</body>

</html>
```

## 属性绑定 与 双向数据绑定

```md
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <title>myvuedemo</title>
  <script src="./vue.js"></script>
</head>

<body>
  <div id="app">
    <div v-bind:title="message">属性绑定</div>
    <div :title="message">属性绑定2</div> 
    <!-- :title= 与 v-bind:title= 等效-->
    <button v-on:click="handle_event">改变title属性</button><br/>
    
    <!-- 双向数据绑定 -->
    <input v-model="content">
    <div>{{content}}</div>
  </div>

  <script>
    var app = new Vue({
      el: '#app',
      data: {
        message: 'Hello Vue!',
        content: ''
      },
      methods: {
        handle_event: function () {
          this.message = 'Hello World'
        }
      }
    })
  </script>
</body>

</html>
```

## 计算属性与侦听器

```md
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <title>myvuedemo</title>
  <script src="./vue.js"></script>
</head>

<body>
  <div id="app">
    <input v-model="firstName"><br>
    <input v-model="lastName">
    <div>全名:{{fullName}}</div>
    <div>更新次数:{{changeCount}}</div>
  </div>

  <script>
    var app = new Vue({
      el: '#app',
      data: {
        firstName: '',
        lastName: '',
        changeCount: 0
        // 计算属性就不能再在data属性里重复写了
      },
      // 计算属性
      computed: {
        fullName: function () {
          return this.firstName + ' ' + this.lastName
        }
      },
      // 侦听器
      watch: {
        fullName: function (new_v, old_v) {
          console.log('旧值:' + old_v + ' ' + '新值:' + new_v)
          this.changeCount += 1
        }
      }
    })
  </script>
</body>

</html>
```

## v-if v-show

注意:打开控制台,查看元素,比较两者不同点

```md
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <title>myvuedemo</title>
  <script src="./vue.js"></script>
</head>

<body>
  <div id="app">
    <button @click="change_flag">显示还是隐藏,这是个问题</button>
    <div v-if="show_flag">Hello</div>
    <div v-show="show_flag">World</div>
  </div>

  <script>
    var app = new Vue({
      el: '#app',
      data: {
        show_flag: true
      },
      methods: {
        change_flag: function () {
          this.show_flag = !this.show_flag
        }
      }
    })
  </script>
</body>

</html>
```

## v-for

见下一小节

## TodoList

```md
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <title>myvuedemo</title>
  <script src="./vue.js"></script>
</head>

<body>
  <div id="app">
    <div>
      <input v-model="input_value" />
      <button @click="submit_value">Add to list</button> <!-- @click="submit_value 等效于 v-on:click="submit_value" -->
    </div>
    <ul>
      <li v-for="(item,index) of item_list" :key="index"> <!-- :key="index" 等效于 v-bind:key="index" -->
        {{item}}
      </li>
    </ul>
  </div>

  <script>
    var app = new Vue({
      el: '#app', 
      data: {
        message: 'Hello Vue!',
        input_value: '',
        item_list: []
      },
      methods: {
        submit_value: function () {
          this.item_list.push(this.input_value)
          this.input_value = ''
        }
      }
    })
  </script>
</body>

</html>
```

## TodoList 组件化

```md
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <title>myvuedemo</title>
  <script src="./vue.js"></script>
</head>

<body>
  <div id="app">
    <div>
      <input v-model="input_value" />
      <button @click="submit_value">Add to list</button>
    </div>
    <ul>
      <!-- 将此处的属性 item 传递给绑定的子组件属性 content -->
      <todo-list v-for="(item,index) of item_list" :key="index" :content="item"></todo-list>
      <todo-item v-for="(item,index) of item_list" :key="index+1000" :content="item"></todo-item>
    </ul>
  </div>

  <script>

    // 全局组件
    Vue.component('todo-list', {
      // 接收外部组件传入的值
      props:['content'],
      template: '<li>{{content}}</li>'
    })
    // 声明局部组件: 需要注册才能使用
    var TodoItem = {
      props:['content'],
      template: '<li>{{content}}</li>'
    }

    var app = new Vue({
      el: '#app',
      data: {
        message: 'Hello Vue!',
        input_value: '',
        item_list: []
      },
      // 注册组件
      components: {
        'todo-item': TodoItem
      },
      methods: {
        submit_value: function () {
          this.item_list.push(this.input_value)
          this.input_value = ''
        }
      }
    })
  </script>
</body>

</html>
```

## TodoList 子组件与父组件的交互

```md
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <title>myvuedemo</title>
  <script src="./vue.js"></script>
</head>

<body>
  <div id="app">
    <div>
      <input v-model="input_value" />
      <button @click="submit_value">Add to list</button>
    </div>
    <ul>
      <!-- 监听 delete_signal 信号,若有则调用 handle_delete 方法-->
      <todo-item v-for="(item,index) of item_list" :key="index" :content="item" :index_t="index"
        @delete_signal="handle_delete">
      </todo-item>
    </ul>
  </div>

  <script>
    // 子组件
    var TodoItem = {
      props: ['content', 'index_t'],
      template: '<li @click="delete_item">{{content}}</li>',
      methods: {
        delete_item: function () {
          console.log(this.index_t)
          // 给外部发送一个信号,名称为 delete_signal
          this.$emit('delete_signal', this.index_t)
        }
      }
    }

    var app = new Vue({
      el: '#app',
      data: {
        message: 'Hello Vue!',
        input_value: '',
        item_list: []
      },
      components: {
        'todo-item': TodoItem
      },
      methods: {
        submit_value: function () {
          this.item_list.push(this.input_value)
          this.input_value = ''
        },
        handle_delete: function (i) {
          console.log(i)
          this.item_list.splice(i, 1)
        }
      }
    })
  </script>
</body>

</html>

```
