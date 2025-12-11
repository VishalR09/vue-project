<template></template>

<script setup>
import { inject, watchEffect } from "vue";
import * as d3 from "d3";

const height = inject("height");
const scales = inject("scales");

watchEffect(() => {
  if (!height || !scales) return;
  scales.y = d3.scaleLinear().domain([0, 100]).range([height, 0]);
});
</script>
