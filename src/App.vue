<template>
  <div id="app">
    <QrReader result="" error="" />
  </div>
</template>

<script>
import QrReader from './components/QrReader'

export default {
  name: 'app',
  components: {
    QrReader
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

.header {
  background-color: black;
  padding: 10px 5px;
  color: white;
  text-transform: uppercase;
  font-weight: bold;
}

.modal-backdrop {
  display: none;
}
</style>
