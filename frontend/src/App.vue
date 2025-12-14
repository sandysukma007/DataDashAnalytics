<template>
  <div id="app">
    <!-- Bisa pakai router-view jika nanti pakai Vue Router -->
    <Dashboard />
  </div>
</template>

<script setup>
import Dashboard from './views/Dashboard.vue'

</script>

<style>
/* global styles, misal font, body, background */
body {
  margin: 0;
  font-family: 'Arial', sans-serif;
  background-color: #f5f5f5;
}
#app {
  min-height: 100vh;
}
</style>
