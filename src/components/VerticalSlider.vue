<template>
  <div
    class="scrollbar vertical"
    :style="{
      left: margin.left + width + 6 + 'px',
      top: margin.top + 'px',
      height: height + 'px',
      width: bar + 'px'
    }"
    @wheel.prevent="onWheel"
  >
    <div class="track">
      <div
        class="thumb"
        :style="{ height: thumbH + 'px', top: thumbTop + 'px' }"
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

const winY = computed(() => scales.y.domain()[1] - scales.y.domain()[0]);

const offsetY = computed({
  get: () => scales.y.domain()[0],
  set: v => {
    const maxOff = 100 - winY.value;
    const clamped = clamp(v, 0, maxOff);
    setScales(scales.x.domain(), [clamped, clamped + winY.value]);
  }
});

const thumbH = computed(() => Math.max(minThumb, (winY.value / 100) * height));
const thumbTop = computed(() => {
  const maxOff = 100 - winY.value;
  const travel = height - thumbH.value;
  return maxOff === 0 ? 0 : (1 - offsetY.value / maxOff) * travel;
});

function onWheel(e) {
  const step = Math.max(1, winY.value / 10);
  const maxOff = 100 - winY.value;
  offsetY.value = clamp(offsetY.value + (e.deltaY > 0 ? -step : +step), 0, maxOff);
}

function startDrag(e) {
  const startPos = e.clientY, startOff = offsetY.value;
  const travel = height - thumbH.value, maxOff = 100 - winY.value;
  function move(ev) {
    const delta = (ev.clientY - startPos) / travel * maxOff;
    offsetY.value = clamp(startOff - delta, 0, maxOff);
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