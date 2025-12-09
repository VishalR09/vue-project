<template>
  <div v-if="zoomEnabled" class="pan-overlay">
    <!-- Horizontal scrollbar -->
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

    <!-- Vertical scrollbar -->
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

const scales = inject("scales"),
  setScales = inject("setScales");
const zoomEnabled = inject("zoomEnabled"),
  width = inject("width"),
  height = inject("height"),
  margin = inject("margin");

const bar = 14,
  minThumb = 24;

// Window sizes
const winX = computed(() => scales.x.domain()[1] - scales.x.domain()[0]);
const winY = computed(() => scales.y.domain()[1] - scales.y.domain()[0]);

// Offsets
const offsetX = computed({
  get: () => scales.x.domain()[0],
  set: (v) =>
    setScales(
      [clamp(v, 0, 100 - winX.value), v + winX.value],
      scales.y.domain()
    ),
});
const offsetY = computed({
  get: () => scales.y.domain()[0],
  set: (v) =>
    setScales(scales.x.domain(), [
      clamp(v, 0, 100 - winY.value),
      v + winY.value,
    ]),
});

// Thumb sizes & positions
const thumbW = computed(() => Math.max(minThumb, (winX.value / 100) * width));
const thumbH = computed(() => Math.max(minThumb, (winY.value / 100) * height));
const thumbLeft = computed(
  () => (offsetX.value / (100 - winX.value)) * (width - thumbW.value)
);
const thumbTop = computed(
  () => (1 - offsetY.value / (100 - winY.value)) * (height - thumbH.value)
); // inverted mapping

// Wheel scroll
function onWheelX(e) {
  const step = Math.max(1, winX.value / 10);
  offsetX.value = clamp(
    offsetX.value + (e.deltaY > 0 ? step : -step),
    0,
    100 - winX.value
  );
}
function onWheelY(e) {
  const step = Math.max(1, winY.value / 10);
  offsetY.value = clamp(
    offsetY.value + (e.deltaY > 0 ? -step : +step),
    0,
    100 - winY.value
  );
}

// Dragging
function startDrag(axis, e) {
  const startPos = axis === "x" ? e.clientX : e.clientY;
  const startOff = axis === "x" ? offsetX.value : offsetY.value;
  const track = axis === "x" ? width : height;
  const thumb = axis === "x" ? thumbW.value : thumbH.value;
  const maxOff = 100 - (axis === "x" ? winX.value : winY.value);
  const travel = track - thumb;

  function move(ev) {
    const curr = axis === "x" ? ev.clientX : ev.clientY;
    const deltaPx = curr - startPos;
    const deltaOff = (deltaPx / travel) * maxOff;
    const newOff = clamp(
      startOff + (axis === "x" ? deltaOff : -deltaOff),
      0,
      maxOff
    );
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

// Utility
function clamp(v, a, b) {
  return Math.min(Math.max(v, a), b);
}
</script>

<style scoped>
.pan-overlay {
  position: absolute;
  top: 0;
  left: 0;
}
.scrollbar {
  position: absolute;
  pointer-events: auto;
}

.track {
  position: absolute;
  width: 100%;
  height: 100%;
  background: #f3f3f3;
  border: 1px solid #ccc;
  border-radius: 8px;
  left: 380px;
  top: 118px; /* ✅ kept as requested */
}

.thumb {
  position: absolute;
  background: #999;
  border-radius: 8px;
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
