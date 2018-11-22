<template>
  <div id="app">
    <TabUnit :width="width"
             :height="height" />
  </div>
</template>

<script>
import Vue from 'vue'
import VueKonva from 'vue-konva'
import TabUnit from './components/TabUnit.vue'

Vue.use(VueKonva)

export default {
  name: 'app',
  components: {
    TabUnit
  },
  data () {
    return {
      width: 400,
      height: 200
    }
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
