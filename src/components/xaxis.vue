<template>
  <g ref="axisRef" :transform="`translate(0, ${height})`"></g>
</template>

<script setup>
import { inject, ref, watchEffect } from "vue";
import * as d3 from "d3";

const scales = inject("scales");
const width = inject("width");
const height = inject("height");
const axisRef = ref(null);

watchEffect(() => {
  if (!axisRef.value || !width) return;

  if (!scales.x) {
    scales.x = d3.scaleTime()
      .domain([new Date(2025,0,1,0,0), new Date(2025,0,1,10,0)])
      .range([0, width]);
  }

  const g = d3.select(axisRef.value);
  g.selectAll("*").remove();

  g.append("g")
    .call(
      d3.axisBottom(scales.x)
        .ticks(d3.timeHour.every(1))
        .tickFormat(d => `${d3.timeFormat("%a")(d)} ${d3.timeFormat("%H:%M")(d)}`)
        .tickSize(10)
    )
    .selectAll("text")
    .attr("transform", "rotate(-45)")
    .style("text-anchor", "end");

  g.append("g")
    .call(
      d3.axisBottom(scales.x)
        .ticks(d3.timeMinute.every(30))
        .tickFormat(() => "")
        .tickSize(6)
    )
    .select(".domain").remove();
});
</script>
