<template>
  <div class="line-Chart">
    <Buttons />
    <svg
      :width="width + margin.left + margin.right"
      :height="height + margin.top + margin.bottom"
    >
      <g :transform="`translate(${margin.left},${margin.top})`">
        <!-- Zoom and Brush -->
        <Zoom ref="zoomRef" />
        <Brush ref="brushRef" />

        <!-- Axes -->
        <Xaxis />
        <Yaxis />

        <!-- Scales -->
        <Xscale />
        <Yscale />

        <!-- Lines -->
        <g clip-path="url(#clip)">
          <Line
            v-for="(seg, i) in segments"
            :key="i"
            :start="seg.start"
            :end="seg.end"
            @drag-segment="moveSegment(seg, $event)"
          />
        </g>
      </g>
      <defs>
        <clipPath id="clip"><rect :width="width" :height="height" /></clipPath>
      </defs>
    </svg>

<!-- // Here for pan // -->
    <Pan />
  </div>
</template>

<script setup>
import { provide, reactive, ref } from "vue";
import * as d3 from "d3";
import Xscale from "./xscale.vue";
import Yscale from "./yscale.vue";
import Xaxis from "./xaxis.vue";
import Yaxis from "./yaxis.vue";
import Line from "./line.vue";
import Brush from "./brush.vue";
import Zoom from "./zoom.vue";
import Buttons from "./buttons.vue";
import Pan from "./pan.vue"; 

// ----------BASIC DATA & LAYOUT-------------
const width = 500, height = 400;
const margin = { top: 20, right: 20, bottom: 40, left: 50 };
const data = [
  { x: 0, y: 10 },
  { x: 10, y: 20 },
  { x: 20, y: 30 },
  { x: 30, y: 40 },
  { x: 40, y: 50 },
  { x: 70, y: 60 },
  { x: 80, y: 90 },
];

// ---SCALES---
const scales = reactive({ x: null, y: null });
function setScales(xDom, yDom) {
  scales.x = d3.scaleLinear().domain(xDom).range([0, width]);
  scales.y = d3.scaleLinear().domain(yDom).range([height, 0]);
}
provide("scales", scales);
provide("width", width);
provide("height", height);
provide("margin", margin);
provide("data", data);
provide("setScales", setScales);

// -------LINE SEGMENTS-------
function MiddleSegments(arr) {
  const segs = [];
  for (let i = 0; i < arr.length - 1; i++) {
    const s = arr[i], e = arr[i + 1], m = { x: s.x, y: e.y };
    segs.push({ start: s, end: m }, { start: m, end: e });
  }
  return segs;
}
const segments = reactive(MiddleSegments(data));
function moveSegment(seg, { dx, dy }) {
  if (seg.start.y === seg.end.y) {
    seg.start.y += dy;
    seg.end.y += dy;
  } else if (seg.start.x === seg.end.x) {
    seg.start.x += dx;
    seg.end.x += dx;
  }
}

// -----------ZOOM & BRUSH--------
const zoomEnabled = ref(false);
const zoomRef = ref(null), brushRef = ref(null);
function toggleZoom() { zoomEnabled.value = !zoomEnabled.value; }
function resetZoom() {
  setScales([0, 100], [100, 0]); 
  zoomRef.value?.reset();
  brushRef.value?.clear();
  zoomEnabled.value = false;
}
provide("zoomEnabled", zoomEnabled);
provide("toggleZoom", toggleZoom);
provide("resetZoom", resetZoom);
</script>

<style scoped>
.line-Chart { user-select: none; }
</style>