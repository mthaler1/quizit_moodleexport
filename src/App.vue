<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    
    <ConvertMoodle />
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import ConvertMoodle from './components/ConvertMoodle.vue'

export default {
  name: 'App',
  components: {
    HelloWorld,
    ConvertMoodle
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
