<template>
  <rect
    ref="overlay"
    :width="width"
    :height="height"
    fill="transparent"
    :pointer-events="zoomEnabled ? 'all' : 'none'"
  />
</template>

<script setup>
import { inject, ref, watch, onMounted } from "vue";
import * as d3 from "d3";

const scales = inject("scales");
const setScales = inject("setScales");
const width = inject("width");
const height = inject("height");
const zoomEnabled = inject("zoomEnabled");

const overlay = ref(null);
let zoomBehavior;


function enableZoom() {
  const el = d3.select(overlay.value);
  el.on(".zoom", null); 

  const x0 = scales.x.copy();
  const y0 = scales.y.copy();

  zoomBehavior = d3.zoom().on("zoom", (e) => {
    setScales(
      e.transform.rescaleX(x0).domain(),
      e.transform.rescaleY(y0).domain()
    );
  });

  el.style("touch-action", "none").call(zoomBehavior);
}



watch(zoomEnabled, (active) => {
  active
    ? enableZoom()
    : d3.select(overlay.value).on(".zoom", null);
});


onMounted(() => {
  if (zoomEnabled?.value) enableZoom();
});


function reset() {
  setScales([0, 100], [0, 100]);
  enableZoom(); 
  d3.select(overlay.value).call(zoomBehavior.transform, d3.zoomIdentity);
}

defineExpose({ reset });
</script>
