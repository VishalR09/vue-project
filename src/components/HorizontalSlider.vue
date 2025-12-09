<template>
  <div
    class="scrollbar horizontal"
    :style="{
      left: margin.left + 'px',
      top: margin.top + height + 6 + 'px',
      width: width + 'px',
      height: bar + 'px'
    }"
    @wheel.prevent="onWheel"
  >
    <div class="track">
      <div
        class="thumb"
        :style="{ width: thumbW + 'px', left: thumbLeft + 'px' }"
        @mousedown.prevent="startDrag($event)"
      />
    </div>
  </div>
</template>

<script setup>
import { inject, computed } from "vue";

const scales = inject("scales");
const setScales = inject("setScales");
const width = inject("width");
const height = inject("height");
const margin = inject("margin");

const bar = 14;
const minThumb = 24;

const winX = computed(() => scales.x.domain()[1] - scales.x.domain()[0]);

const offsetX = computed({
  get: () => scales.x.domain()[0],
  set: v => {
    const maxOff = 100 - winX.value;
    const clamped = clamp(v, 0, maxOff);
    setScales([clamped, clamped + winX.value], scales.y.domain());
  }
});

const thumbW = computed(() => Math.max(minThumb, (winX.value / 100) * width));
const thumbLeft = computed(() => {
  const maxOff = 100 - winX.value;
  const travel = width - thumbW.value;
  return maxOff === 0 ? 0 : (offsetX.value / maxOff) * travel;
});

function onWheel(e) {
  const step = Math.max(1, winX.value / 10);
  const maxOff = 100 - winX.value;
  offsetX.value = clamp(offsetX.value + (e.deltaY > 0 ? step : -step), 0, maxOff);
}

function startDrag(e) {
  const startPos = e.clientX, startOff = offsetX.value;
  const travel = width - thumbW.value, maxOff = 100 - winX.value;
  function move(ev) {
    const delta = (ev.clientX - startPos) / travel * maxOff;
    offsetX.value = clamp(startOff + delta, 0, maxOff);
  }
  function up() {
    document.removeEventListener("mousemove", move);
    document.removeEventListener("mouseup", up);
  }
  document.addEventListener("mousemove", move);
  document.addEventListener("mouseup", up);
}

function clamp(v, a, b) { return Math.min(Math.max(v, a), b); }
</script>