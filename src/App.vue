<template>
  <div id="app">
    <Index />
  </div>
</template>

<script>
import Index from './components/Index.vue'

export default {
  name: 'App',
  components: {
    Index
  }
}
</script>

<style>
html,body {
  margin: 0;
  height: 100%;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: lightgray;
  height: 100%;
}
</style>
