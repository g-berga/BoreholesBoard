<template>
  <div id="app">
    <b-navbar :sticky="true" class="navbar" type="dark" variant="dark">
      <b-navbar-brand>Italian Boreholes | Board</b-navbar-brand>
    </b-navbar>
    <side-bar :visible='true'>
      <template slot="links">
        <sidebar-item
          :link="{
            name: 'Dashboard',
            icon: 'ni ni-shop text-primary',
            path: '/dashboard'
          }">
        </sidebar-item>
        <sidebar-item
          :link="{
            name: 'Icons',
            icon: 'ni ni-planet text-orange',
            path: '/icons'
          }">
        </sidebar-item>
      </template>
    </side-bar>
    <div>
    <MapWithCharts></MapWithCharts>
    </div>
  </div>
</template>

<script>

import MapWithCharts from '@/components/Root.vue'

export default {
  name: 'App',
  components: {
    MapWithCharts
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  background-color: #495057;
}
</style>
