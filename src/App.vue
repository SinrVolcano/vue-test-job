<template>
  <div id="app">
    <DynamicForm api="http://www.mocky.io/v2" id="5c65780e3300003b1db99dfd" />
  </div>
</template>

<script>
import DynamicForm from './components/DynamicForm.vue'

export default {
  name: 'app',
  components: {
    DynamicForm
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
