<template>
  <div class="btn-container">
    <button @click="toggleZoom" :class="{ active: zoomEnabled }">Zoom</button>
    <button @click="resetZoom">Reset</button>
  </div>
</template>

<script setup>
import { inject } from "vue";
const toggleZoom = inject("toggleZoom");
const resetZoom = inject("resetZoom");
const zoomEnabled = inject("zoomEnabled");
</script>

<style scoped>
.btn-container {
  display: flex;
  gap: 8px;
  justify-content: center;
  margin-bottom: 8px;
}
button {
  padding: 6px 10px;
  cursor: pointer;
}
button.active {
  background: green;
  color: white;
}
</style>
