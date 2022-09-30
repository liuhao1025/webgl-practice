<script setup lang="ts">
import FactoryDemo from "./components/FactoryDemo.vue";
import WebGLVue from "./components/WebGL.vue";
</script>

<template>
  <!-- <FactoryDemo /> -->
  <WebGLVue />
</template>

<style>
html,
body {
  margin: 0;
  padding: 0;
  height: 100%;
  overflow: hidden;
}
#app {
  height: 100%;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
