<template>
  <div class="header-actions">
    <button class="btn btn-secondary" @click="$emit('export')">📥 Export</button>
    <button class="btn btn-primary" @click="$emit('refresh')">🔄 Refresh</button>
  </div>
</template>

<script setup></script>
<style src="../assets/css/style.css"></style>
