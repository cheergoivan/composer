<template>
  <v-app>
    <v-toolbar app>
      <v-toolbar-title class="headline text-uppercase">
        <span>Vuetify</span>
        <span class="font-weight-light">MATERIAL DESIGN</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn flat
             href="https://github.com/vuetifyjs/vuetify/releases/latest"
             target="_blank">
        <span class="mr-2">Latest Release</span>
      </v-btn>
    </v-toolbar>

    <v-content>
      <tab-unit :width="width"
                :height="height"></tab-unit>
    </v-content>
  </v-app>
</template>

<script>
import TabUnit from './components/TabUnit'

export default {
  name: 'App',
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
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
