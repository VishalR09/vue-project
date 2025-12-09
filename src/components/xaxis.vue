<template>
  <g ref="axisRef" v-if="scales.x" :transform="`translate(0, ${height})`"></g>
</template>
 
<script setup>
import { inject, ref, watchEffect } from 'vue'
import * as d3 from 'd3'
 
const scales = inject('scales')
const height = inject('height')
const axisRef = ref(null)
 
watchEffect(() => {
  if (scales?.x && axisRef.value) {
    d3.select(axisRef.value)
    .call(d3.axisBottom(scales.x))
  }
})
</script>