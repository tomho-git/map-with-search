<template>
  <div id="app">
    <div class="map"><Map/></div>
    <div class="control"><Control/></div>
  </div>
</template>

<script>
import Map from "./components/mapToShow.vue"
import Control from "./components/controlPanel.vue"

export default {
  name: 'App',
  components: {
    Map, 
    Control
  }
}
</script>

<style scoped>
#app{
  height: 100vh; 
  width: 100vw; 
  display: flex
}
html {
  height: 100%;
}
body {
  height: 100%;
}
.map{
  width: 50%; 
  height: 100%;
}
.control{
  width: 50%; 
  height: 100%
}
</style>
