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