<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <Statement model="student" id="11"/>
  </div>
</template>

<script>
import Statement from './components/Statement.vue';

export default {
  name: 'App',
  components: {
    // eslint-disable-next-line vue/no-unused-components
    Statement
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
