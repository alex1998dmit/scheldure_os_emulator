<template>
  <div id="app" class="container">
    <div class="row">
    </div>
    <Schedule></Schedule>
  </div>
</template>

<script>
import Schedule from "./components/Schedule"

export default {
  name: 'app',
  components: {
    Schedule
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
