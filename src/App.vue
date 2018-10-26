<template>
  <div id="app">
    <d3-map />
  </div>
</template>

<script>
import D3Map from './components/map.vue'

export default {
  name: 'app',
  components: {
    D3Map
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
}
html, body {
  margin: 0
}
#app {
  height: 100vh;
  width: 100vw;
}
</style>
