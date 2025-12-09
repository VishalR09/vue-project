<template></template>
<script setup>
import { inject, watchEffect } from 'vue'
import * as d3 from 'd3'
 
const width = inject('width')
const scales = inject('scales')
 
watchEffect(() => {
  if (width && scales) {
    scales.x = d3.scaleLinear()
    .domain([0, 100])
    .range([0, width])
  }
})
</script>
 
 

  <!-- <template></template>
<script setup>
import { inject, watchEffect } from 'vue'
import * as d3 from 'd3'

const width = inject('width')
const scales = inject('scales')

// Initialize only once
watchEffect(() => {
  if (width && scales && !scales.x) {
    scales.x = d3.scaleLinear().domain([0, 100]).range([0, width])
  }
})
</script> -->