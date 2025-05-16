<template>
  <div id="app">
    <SlidingPuzzle />
  </div>
</template>

<script>
import SlidingPuzzle from './components/SlidingPuzzle.vue'

export default {
  name: 'App',
  components: {
    SlidingPuzzle
  }
}
</script>

<style>
#app {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin: 0;
  padding: 20px;
  background-color: #f5f5f5;
  min-height: 100vh;
}
</style>