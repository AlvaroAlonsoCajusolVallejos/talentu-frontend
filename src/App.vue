<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
  <Usuarios/>
</template>

<script>
import Usuarios from './components/Usuarios.vue'

export default {
  name: 'App',
  components: {
    Usuarios
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
