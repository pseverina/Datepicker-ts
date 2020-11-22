<template>
  <div id="app">
    <h1>Enjoy the simple datapicker</h1>
    <datapicker></datapicker>
  </div>
</template>

<script>
import Datapicker from './components/Datapicker'

export default {
  name: 'App',
  components: {
    Datapicker
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
  height: 100%;
  margin: 0 auto;
}
</style>
