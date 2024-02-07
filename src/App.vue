<template>
  <img alt="Logs de AbsysNet" src="./assets/e-log.png" width="300px">
  <CheckItems msg="Visor de los archivos obtenidos, y su estado, en AbsysNet"/>
</template>

<script>
import CheckItems from './components/CheckItems.vue'

export default {
  name: 'App',
  components: {
    CheckItems
  }
}
</script>

<style>
body {
  margin: 0;
  font-family: var(--bs-body-font-family);
  font-size: var(--bs-body-font-size);
  font-weight: var(--bs-body-font-weight);
  line-height: var(--bs-body-line-height);
  color: var(--bs-body-color);
  text-align: var(--bs-body-text-align);
  background-color: var(--bs-body-bg);
  -webkit-text-size-adjust: 100%;
  -webkit-tap-highlight-color: transparent;
  background-color: #f4f6fc;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 40px;
}
</style>
