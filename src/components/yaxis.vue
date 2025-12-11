<template>
  <g ref="axisRef" v-if="scales?.y"></g>
</template>

<script setup>
import { inject, ref, watchEffect } from "vue";
import * as d3 from "d3";

const scales = inject("scales");
const axisRef = ref(null);

watchEffect(() => {
  if (!scales?.y || !axisRef.value) return;
  d3.select(axisRef.value).call(d3.axisLeft(scales.y).tickFormat(v => `${v}°C`));
});
</script>
