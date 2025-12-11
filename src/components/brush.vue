<template>
  <g ref="brush"></g>
</template>

<script setup>
import { inject, ref, watch, onMounted } from "vue";
import * as d3 from "d3";

const scales = inject("scales");
const width = inject("width");
const height = inject("height");
const setScales = inject("setScales");
const zoomEnabled = inject("zoomEnabled");

const brush = ref(null);
const MIN_BRUSH_SIZE = 4;

function enableBrush() {
  const g = d3.select(brush.value);
  g.selectAll("*").remove();

  g.append("rect")
    .attr("width", width)
    .attr("height", height)
    .attr("fill", "transparent")
    .style("cursor", "zoom-in")
    .on("mousedown", (e) => {
      e.preventDefault();
      const start = d3.pointer(e, brush.value);

      const box = g
        .append("rect")
        .attr("x", start[0])
        .attr("y", start[1])
        .attr("fill", "steelblue")
        .attr("fill-opacity", 0.25)
        .attr("stroke", "steelblue");

      const onMove = (ev) => {
        const [x, y] = d3.pointer(ev, brush.value);
        box
          .attr("x", Math.min(start[0], x))
          .attr("y", Math.min(start[1], y))
          .attr("width", Math.abs(x - start[0]))
          .attr("height", Math.abs(y - start[1]));
      };

      const onUp = (ev) => {
        const [x, y] = d3.pointer(ev, brush.value);
        const w = Math.abs(x - start[0]),
          h = Math.abs(y - start[1]);
        if (w >= MIN_BRUSH_SIZE && h >= MIN_BRUSH_SIZE) {
          const minPx = Math.min(start[0], x),
            maxPx = Math.max(start[0], x);
          const minY = Math.min(start[1], y),
            maxY = Math.max(start[1], y);
          setScales(
            [scales.x.invert(minPx), scales.x.invert(maxPx)],
            [scales.y.invert(maxY), scales.y.invert(minY)]
          );
        }
        box.remove();
        d3.select(window).on("mousemove.brush", null).on("mouseup.brush", null);
      };

      d3.select(window).on("mousemove.brush", onMove).on("mouseup.brush", onUp);
    });
}

function disableBrush() {
  d3.select(brush.value).selectAll("*").remove();
  d3.select(window).on("mousemove.brush", null).on("mouseup.brush", null);
}

watch(zoomEnabled, (v) => (v ? enableBrush() : disableBrush()));
onMounted(() => {
  if (zoomEnabled?.value) enableBrush();
});

defineExpose({ clear: disableBrush });
</script>
