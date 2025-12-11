<template>
  <line
    ref="dragArea"
    :x1="coord.x1"
    :y1="coord.y1"
    :x2="coord.x2"
    :y2="coord.y2"
    stroke="transparent"
    stroke-width="30"
    stroke-linecap="round"
    class="drag-area"
  />
  <line
    :x1="coord.x1"
    :y1="coord.y1"
    :x2="coord.x2"
    :y2="coord.y2"
    stroke="darkblue"
    stroke-width="2"
    stroke-linecap="round"
    class="line"
  />
</template>

<script setup>
import { inject, computed, onMounted, ref } from "vue";
import * as d3 from "d3";

const scales = inject("scales");

const props = defineProps({
  start: { type: Object, required: true },
  end: { type: Object, required: true },
});

const emit = defineEmits(["drag-segment"]);
const dragArea = ref(null);

onMounted(() => {
  d3.select(dragArea.value).call(
    d3.drag().on("drag", (ev) => {
      if (!scales?.xMinutes || !scales?.y) return;

      const dxMinutes =
        scales.xMinutes.invert(ev.dx) - scales.xMinutes.invert(0);

      const dy = scales.y.invert(ev.dy) - scales.y.invert(0);

      emit("drag-segment", { dx: dxMinutes, dy });
    })
  );
});

const coord = computed(() => ({
  x1: scales.x(props.start.time),
  y1: scales.y(props.start.value),
  x2: scales.x(props.end.time),
  y2: scales.y(props.end.value),
}));
</script>

<style scoped>
.line:hover {
  stroke: red;
  cursor: pointer;
}
.drag-area {
  cursor: grab;
}
</style>
