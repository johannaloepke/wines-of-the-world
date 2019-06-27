<template>
  <div id="app">
    <WorldMap msg="Welcome to Wines of the World"/>
  </div>
</template>

<script>
import WorldMap from './components/WorldMap.vue'

export default {
  name: 'app',
  components: {
    WorldMap
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
</style>
