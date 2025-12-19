<template>
  <div class="save-btn-container">
    <button
      v-for="(s, i) in saves"
      :key="i"
      :class="{ active: activeIndex === i }"
      @click="$emit('select', i)"
    >
      <div class="label">{{ s.label }}</div>
      <div class="time">{{ s.time }}</div>
    </button>
  </div>
</template>

<script setup>
defineProps({
  saves: { type: Array, required: true },
  activeIndex: { type: Number, required: true },
});

defineEmits(["select"]);
</script>

<style scoped>
.save-btn-container {
  display: flex;
  gap: 8px;
  justify-content: center;
  margin-top: 10px;
}

button {
  padding: 6px 10px;
  cursor: pointer;
  min-width: 90px;
  text-align: center;
}

button.active {
  background: #4595e5;
  color: black;
}

.label {
  font-weight: 550;
  font-size: 13px;
}

.time {
  font-size: 12px;
}
</style>
