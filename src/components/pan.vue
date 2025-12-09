<template>
  <div v-if="zoomEnabled" class="pan-overlay">
    <!-- Horizontal scrollbar (shortened to avoid vertical bar and track offset) -->
    <div
      class="scrollbar horizontal"
      :style="{ 
        left: margin.left + 'px',
        top: margin.top + height + 'px',
        width: (width - bar - trackOffsetX) + 'px',
        height: bar + 'px'
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

    <!-- Vertical scrollbar (shortened to avoid horizontal bar and track offset) -->
    <div
      class="scrollbar vertical"
      :style="{ 
        left: margin.left + width + 'px',
        top: margin.top + 'px',
        height: (height - bar - trackOffsetY) + 'px',
        width: bar + 'px'
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

const scales = inject("scales"), setScales = inject("setScales");
const zoomEnabled = inject("zoomEnabled"), width = inject("width"), height = inject("height"), margin = inject("margin");

// Bar thickness and track offsets (must match CSS below)
const bar = 14;
const trackOffsetX = 380; // horizontal track is shifted right by 380px
const trackOffsetY = 118; // vertical track is shifted down by 118px

// Longer thumbs while keeping behavior the same
const minThumb = 80;

// Window sizes (percent domain window)
const winX = computed(() => scales.x.domain()[1] - scales.x.domain()[0]);
const winY = computed(() => scales.y.domain()[1] - scales.y.domain()[0]);

// Offsets (top-left of visible domain)
const offsetX = computed({
  get: () => scales.x.domain()[0],
  set: v => setScales([clamp(v, 0, 100 - winX.value), v + winX.value], scales.y.domain())
});
const offsetY = computed({
  get: () => scales.y.domain()[0],
  set: v => setScales(scales.x.domain(), [clamp(v, 0, 100 - winY.value), v + winY.value])
});

// Effective track sizes (reduce by other bar and the track CSS offsets)
const trackW = computed(() => Math.max(0, width - bar - trackOffsetX));
const trackH = computed(() => Math.max(0, height - bar - trackOffsetY));

// Longer thumbs: bigger min + moderate scaling
const thumbW = computed(() => Math.max(minThumb, (winX.value / 100) * trackW.value * 1.8));
const thumbH = computed(() => Math.max(minThumb, (winY.value / 100) * trackH.value * 1.8));

// Thumb positions within effective travel area
const thumbLeft = computed(() => {
  const travel = Math.max(0, trackW.value - thumbW.value);
  return (offsetX.value / (100 - winX.value)) * travel;
});
const thumbTop = computed(() => {
  const travel = Math.max(0, trackH.value - thumbH.value);
  return (1 - offsetY.value / (100 - winY.value)) * travel; // inverted vertical mapping
});

// Wheel scroll (unchanged behavior)
function onWheelX(e) {
  const step = Math.max(1, winX.value / 10);
  offsetX.value = clamp(offsetX.value + (e.deltaY > 0 ? step : -step), 0, 100 - winX.value);
}
function onWheelY(e) {
  const step = Math.max(1, winY.value / 10);
  offsetY.value = clamp(offsetY.value + (e.deltaY > 0 ? -step : +step), 0, 100 - winY.value);
}

// Dragging (uses effective track sizes so thumbs stay within bounds)
function startDrag(axis, e) {
  const startPos = axis === "x" ? e.clientX : e.clientY;
  const startOff = axis === "x" ? offsetX.value : offsetY.value;
  const track = axis === "x" ? trackW.value : trackH.value;
  const thumb = axis === "x" ? thumbW.value : thumbH.value;
  const maxOff = 100 - (axis === "x" ? winX.value : winY.value);
  const travel = Math.max(1, track - thumb); // avoid division by 0

  function move(ev) {
    const curr = axis === "x" ? ev.clientX : ev.clientY;
    const deltaPx = curr - startPos;
    const deltaOff = (deltaPx / travel) * maxOff;
    const newOff = clamp(startOff + (axis === "x" ? deltaOff : -deltaOff), 0, maxOff);
    if (axis === "x") offsetX.value = newOff; else offsetY.value = newOff;
  }
  function up() {
    document.removeEventListener("mousemove", move);
    document.removeEventListener("mouseup", up);
  }
  document.addEventListener("mousemove", move);
  document.addEventListener("mouseup", up);
}

// Utility
function clamp(v, a, b) { return Math.min(Math.max(v, a), b); }
</script>

<style scoped>
.pan-overlay { position: absolute; top: 0; left: 0; }
.scrollbar { position: absolute; pointer-events: auto; overflow: hidden; }

/* Track remains offset as requested */
.track {
  position: absolute;
  width: 100%;
  height: 100%;
  background: #f3f3f3;
  border: 1px solid #ccc;
  border-radius: 8px;
  left: 380px; /* keep this line */
  top: 118px;  /* keep this line */
}

.thumb { position: absolute; background: #999; border-radius: 8px; cursor: pointer; }
.scrollbar.horizontal .thumb { top: 0; height: 100%; }
.scrollbar.vertical .thumb { left: 0; width: 100%; }
</style>