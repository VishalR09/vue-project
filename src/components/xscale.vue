<template></template>

<script setup>
import { inject, watchEffect } from "vue";
import * as d3 from "d3";

const width = inject("width");
const scales = inject("scales");

watchEffect(() => {
  if (!width || !scales) return;
  scales.xMinutes = d3.scaleLinear().domain([0, 600]).range([0, width]);
});
</script>
