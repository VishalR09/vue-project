<template>
  <div v-if="zoomEnabled" class="pan-overlay">
    <div
      class="scrollbar horizontal"
      :style="{
        left: margin.left + 'px',
        top: margin.top + height + 'px',
        width: width + 'px',
        height: bar + 'px',
      }"
      @wheel.prevent="onWheelX"
    >
      <div class="track">
        <div
          class="thumb"
          :style="{ width: thumbW + 'px', left: thumbLeft + 'px' }"
          @mousedown.prevent="startDrag('x', $event)"
        />
      </div>
    </div>

    <div
      class="scrollbar vertical"
      :style="{
        left: margin.left + width + 'px',
        top: margin.top + 'px',
        height: height + 'px',
        width: bar + 'px',
      }"
      @wheel.prevent="onWheelY"
    >
      <div class="track">
        <div
          class="thumb"
          :style="{ height: thumbH + 'px', top: thumbTop + 'px' }"
          @mousedown.prevent="startDrag('y', $event)"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { inject, computed } from "vue";

const scales = inject("scales");
const setScales = inject("setScales");
const zoomEnabled = inject("zoomEnabled");
const width = inject("width");
const height = inject("height");
const margin = inject("margin");
const fullXStart = inject("fullXStart");
const fullXMinutes = inject("fullXMinutes");
const fullYRange = inject("fullYRange");

const bar = 8;
const minThumb = 20;

const winX = computed(() => {
  const dom = scales.x.domain();
  return (dom[1].getTime() - dom[0].getTime()) / 60000;
});
const winY = computed(() => {
  const dom = scales.y.domain();
  return dom[1] - dom[0];
});

const offsetX = computed({
  get: () => (scales.x.domain()[0].getTime() - fullXStart.getTime()) / 60000,
  set: (v) => {
    const clamped = Math.min(Math.max(v, 0), fullXMinutes - winX.value);
    const newStart = new Date(fullXStart.getTime() + clamped * 60000);
    const newEnd = new Date(newStart.getTime() + winX.value * 60000);
    setScales([newStart, newEnd], scales.y.domain());
  },
});
const offsetY = computed({
  get: () => scales.y.domain()[0],
  set: (v) => {
    const clamped = Math.min(Math.max(v, 0), fullYRange - winY.value);
    setScales(scales.x.domain(), [clamped, clamped + winY.value]);
  },
});

const thumbW = computed(() =>
  Math.max(minThumb, (winX.value / fullXMinutes) * width)
);
const thumbH = computed(() =>
  Math.max(minThumb, (winY.value / fullYRange) * height)
);

const thumbLeft = computed(() => {
  const denom = fullXMinutes - winX.value || 1;
  return (offsetX.value / denom) * (width - thumbW.value);
});
const thumbTop = computed(() => {
  const denom = fullYRange - winY.value || 1;
  return (1 - offsetY.value / denom) * (height - thumbH.value);
});

function onWheelX(e) {
  const step = Math.max(1, winX.value / 10);
  offsetX.value = offsetX.value + (e.deltaY > 0 ? step : -step);
}
function onWheelY(e) {
  const step = Math.max(1, winY.value / 10);
  offsetY.value = offsetY.value + (e.deltaY > 0 ? -step : +step);
}

function startDrag(axis, e) {
  const startPos = axis === "x" ? e.clientX : e.clientY;
  const startOff = axis === "x" ? offsetX.value : offsetY.value;
  const track = axis === "x" ? width : height;
  const thumb = axis === "x" ? thumbW.value : thumbH.value;
  const maxOff =
    axis === "x" ? fullXMinutes - winX.value : fullYRange - winY.value;
  const travel = track - thumb;

  function move(ev) {
    const curr = axis === "x" ? ev.clientX : ev.clientY;
    const deltaPx = curr - startPos;
    const deltaOff = (deltaPx / travel) * maxOff;
    const newOff = startOff + (axis === "x" ? deltaOff : -deltaOff);
    if (axis === "x") offsetX.value = newOff;
    else offsetY.value = newOff;
  }
  function up() {
    document.removeEventListener("mousemove", move);
    document.removeEventListener("mouseup", up);
  }
  document.addEventListener("mousemove", move);
  document.addEventListener("mouseup", up);
}
</script>

<style scoped>
.pan-overlay {
  position: absolute;
  top: 0;
  left: 0;
  pointer-events: none;
}
.scrollbar {
  position: absolute;
  pointer-events: auto;
}
.track {
  position: relative;
  width: 100%;
  height: 100%;
  background: #f3f3f3;
  border: 1px solid #ccc;
  border-radius: 8px;
  left: 380px;
  top: 108px;
}
.thumb {
  position: absolute;
  background: #999;
  border-radius: 6px;
  cursor: pointer;
}
.scrollbar.horizontal .thumb {
  top: 0;
  height: 100%;
}
.scrollbar.vertical .thumb {
  left: 0;
  width: 100%;
}
</style>
