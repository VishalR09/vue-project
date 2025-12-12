<template>
  <div class="line-Chart">
    <Buttons />
    <svg
      :width="width + margin.left + margin.right"
      :height="height + margin.top + margin.bottom"
    >
      <g :transform="`translate(${margin.left},${margin.top})`">
        <Xscale />
        <Yscale />
        <Xaxis />
        <Yaxis />

        <g clip-path="url(#clip)">
          <Line
            v-for="(seg, i) in segments"
            :key="i"
            :start="seg.start"
            :end="seg.end"
            @drag-segment="moveSegment(seg, $event)"
          />
        </g>

        <Zoom ref="zoomRef" />
        <Brush ref="brushRef" />
      </g>

      <defs>
        <clipPath id="clip"><rect :width="width" :height="height" /></clipPath>
      </defs>
    </svg>

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


const width = 500, height = 400;
const margin = { top: 20, right: 60, bottom: 80, left: 120 };


const fullXStart = new Date(2025, 0, 1, 0, 0);
const fullXMinutes = 600; // 10 hours


const data = [
  { time: new Date(2025, 0, 1, 0, 0), value: 10 },
  { time: new Date(2025, 0, 1, 2, 0), value: 20 },
  { time: new Date(2025, 0, 1, 4, 0), value: 30 },
  { time: new Date(2025, 0, 1, 6, 0), value: 40 },
  { time: new Date(2025, 0, 1, 8, 0), value: 50 },
  { time: new Date(2025, 0, 1, 9, 0), value: 60 },
  { time: new Date(2025, 0, 1, 10, 0), value: 80 },
];


const scales = reactive({ x: null, xMinutes: null, y: null });

function setScales(xDomDates, yDom) {
  scales.x = d3.scaleTime().domain(xDomDates).range([0, width]);
  scales.xMinutes = d3.scaleLinear().domain([0, fullXMinutes]).range([0, width]);
  scales.y = d3.scaleLinear().domain(yDom).range([height, 0]);
}

provide("scales", scales);
provide("width", width);
provide("height", height);
provide("margin", margin);
provide("data", data);
provide("setScales", setScales);

provide("fullXStart", fullXStart);
provide("fullXMinutes", fullXMinutes);
provide("fullYRange", 100);


const zoomEnabled = ref(false);
const zoomRef = ref(null);
const brushRef = ref(null);

function toggleZoom() { zoomEnabled.value = !zoomEnabled.value; }

function resetZoom() {

  const xExtent = d3.extent(data, (d) => d.time);
  const yExtent = d3.extent(data, (d) => d.value);
  const pad = 0.1 * (yExtent[1] - yExtent[0]);
  setScales(xExtent, [yExtent[0] - pad, yExtent[1] + pad]);


  if (zoomRef.value && typeof zoomRef.value.reset === "function") {
    zoomRef.value.reset();
  }
  if (brushRef.value && typeof brushRef.value.clear === "function") {
    brushRef.value.clear();
  }

  zoomEnabled.value = false;
}

provide("zoomEnabled", zoomEnabled);
provide("toggleZoom", toggleZoom);
provide("resetZoom", resetZoom);
provide("zoomRef", zoomRef);
provide("brushRef", brushRef);


const xExtent = d3.extent(data, (d) => d.time);
const yExtent = d3.extent(data, (d) => d.value);
const pad = 0.1 * (yExtent[1] - yExtent[0]);
setScales(xExtent, [yExtent[0] - pad, yExtent[1] + pad]);


function MiddleSegments(arr) {
  const segs = [];
  for (let i = 0; i < arr.length - 1; i++) {
    const s = arr[i], e = arr[i + 1];
    const m = { time: s.time, value: e.value };
    segs.push({ start: s, end: m }, { start: m, end: e });
  }
  return segs;
}
const segments = reactive(MiddleSegments(data));


function moveSegment(seg, { dx, dy }) {
  if (seg.start.value === seg.end.value) {
    seg.start.value += dy;
    seg.end.value += dy;
    return;
  }
  if (seg.start.time.getTime() === seg.end.time.getTime()) {
    const shiftMs = dx * 60000;
    seg.start.time = new Date(seg.start.time.getTime() + shiftMs);
    seg.end.time = new Date(seg.end.time.getTime() + shiftMs);
  }
}
</script>

<style scoped>
.line-Chart {
  user-select: none;
  position: relative;
}
</style>
