<template>
  <FaceScan />
</template>

<script>
//import HelloWorld from './components/HelloWorld.vue'
import FaceScan from './components/FaceScan.vue'

export default {
  name: 'App',
  components: {
    FaceScan
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
