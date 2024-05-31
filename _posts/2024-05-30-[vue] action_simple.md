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

[开始](../doc/appendix/vue_action_1.html)

## 数据,事件与方法

[数据,事件与方法](../doc/appendix/vue_action_2.html)

## 属性绑定 与 双向数据绑定

[属性绑定 与 双向数据绑定](../doc/appendix/vue_action_3.html)

## 计算属性与侦听器

[计算属性与侦听器](../doc/appendix/vue_action_4.html)

## v-if v-show

注意:打开控制台,查看元素,比较两者不同点

[v-if v-show](../doc/appendix/vue_action_5.html)

## v-for

见下一小节

## TodoList

[TodoList](../doc/appendix/vue_action_6.html)

## TodoList 组件化

[TodoList 组件化](../doc/appendix/vue_action_7.html)

## TodoList 子组件与父组件的交互

[子组件与父组件的交互](../doc/appendix/vue_action_8.html)

## vue-cli 版本的 TodoList

> vue create vue-todolist

App.vue

```vue
<template>
  <div id="app">
    <div>
      <input v-model="input_value" />
      <button @click="submit_value">Add to list</button>
    </div>
    <ul>
      <!-- 监听 delete_signal 信号,若有则调用 handle_delete 方法-->
      <todo-item v-for="(item, index) of item_list" :key="index" :content="item" :index_t="index"
        @delete_signal="handle_delete">
      </todo-item>
    </ul>
  </div>
</template>

<script>
import TodoItem from './components/TodoItem.vue'

export default {
  name: 'App',
  data() {
    return {
      message: 'Hello Vue!',
      input_value: '',
      item_list: []
    }
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
}
</script>
```

TodoItem.vue

```vue
<template>
  <li @click="delete_item">{{ content }}</li>
</template>

<script>
export default {
  name: 'TodoItem',
  props: ['content', 'index_t'],
  methods: {
    delete_item: function () {
      console.log(this.index_t)
      // 给外部发送一个信号,名称为 delete_signal
      this.$emit('delete_signal', this.index_t)
    }
  }
}
</script>
```

