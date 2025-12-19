<template>
  <div class="line-chart">
    <!-- Existing buttons (Zoom / Reset / Save) -->
    <Buttons />

    <svg
      :width="width + margin.left + margin.right"
      :height="height + margin.top + margin.bottom"
    >
      <g :transform="`translate(${margin.left},${margin.top})`">
        <Xaxis />
        <Yaxis />

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

        <Zoom ref="zoomRef" />
        <Brush ref="brushRef" />
      </g>

      <defs>
        <clipPath id="clip">
          <rect :width="width" :height="height" />
        </clipPath>
      </defs>
    </svg>

    <Pan />
  </div>
</template>

<script setup>
import { provide, reactive, ref, watch } from "vue";
import * as d3 from "d3";

import Xaxis from "./xaxis.vue";
import Yaxis from "./yaxis.vue";
import Line from "./line.vue";
import Brush from "./brush.vue";
import Zoom from "./zoom.vue";
import Buttons from "./buttons.vue";
import Pan from "./pan.vue";

const emit = defineEmits(["save"]);

const props = defineProps({
  data: { type: Array, required: true },
});

const width = 500;
const height = 400;
const margin = { top: 20, right: 60, bottom: 80, left: 120 };

const fullXStart = new Date(2025, 0, 1, 0, 0);
const fullXMinutes = 600;
const fullYRange = 100;

const points = reactive([]);
const segments = reactive([]);

function buildSegments() {
  segments.length = 0;
  for (let i = 0; i < points.length - 1; i++) {
    const s = points[i];
    const e = points[i + 1];
    const m = { time: s.time, value: e.value };
    segments.push({ start: s, end: m }, { start: m, end: e });
  }
}

function loadData(data) {
  points.length = 0;
  data.forEach((d) => {
    points.push({
      time: new Date(d.time),
      value: d.value,
    });
  });
  buildSegments();
}

loadData(props.data);

watch(
  () => props.data,
  (v) => loadData(v),
  { deep: true }
);

const scales = reactive({ x: null, y: null });

function setScales(xDom, yDom) {
  scales.x = d3.scaleTime().domain(xDom).range([0, width]);
  scales.y = d3.scaleLinear().domain(yDom).range([height, 0]);
}

const xExtent = d3.extent(props.data, (d) => d.time);
const yExtent = d3.extent(props.data, (d) => d.value);
const pad = 0.1 * (yExtent[1] - yExtent[0]);

setScales(xExtent, [yExtent[0] - pad, yExtent[1] + pad]);

provide("scales", scales);
provide("width", width);
provide("height", height);
provide("margin", margin);
provide("setScales", setScales);
provide("fullXStart", fullXStart);
provide("fullXMinutes", fullXMinutes);
provide("fullYRange", fullYRange);

function moveSegment(seg, { dx, dy }) {
  /* Vertical segment → move Y */
  if (seg.start.value === seg.end.value) {
    seg.start.value += dy;
    seg.end.value += dy;
    return;
  }

  /* Horizontal segment → move X */
  if (seg.start.time.getTime() === seg.end.time.getTime()) {
    seg.start.time = new Date(seg.start.time.getTime() + dx);
    seg.end.time = new Date(seg.end.time.getTime() + dx);
  }
}

const zoomEnabled = ref(false);
const zoomRef = ref(null);
const brushRef = ref(null);

function toggleZoom() {
  zoomEnabled.value = !zoomEnabled.value;
}

function resetZoom() {
  setScales(xExtent, [yExtent[0] - pad, yExtent[1] + pad]);
  zoomRef.value?.reset?.();
  brushRef.value?.clear?.();
  zoomEnabled.value = false;
}

provide("zoomEnabled", zoomEnabled);
provide("toggleZoom", toggleZoom);
provide("resetZoom", resetZoom);
provide("zoomRef", zoomRef);
provide("brushRef", brushRef);

function saveData() {
  emit(
    "save",
    points.map((p) => ({
      time: new Date(p.time),
      value: p.value,
    }))
  );
}

provide("saveData", saveData);

defineExpose({ loadData });
</script>

<style scoped>
.line-chart {
  user-select: none;
  position: relative;
}
</style>
