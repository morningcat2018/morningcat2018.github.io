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