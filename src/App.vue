<template>
  <div id="app">
    <h3>Line graph</h3>

    <Chart :data="states[activeIndex]" @save="saveState" />

    <SaveButtons
      :saves="saves"
      :activeIndex="activeIndex"
      @select="selectState"
    />
  </div>
</template>

<script setup>
import { ref } from "vue";
import Chart from "./components/Chart.vue";
import SaveButtons from "./components/saveButtons.vue";
import rawData from "./server/chartData.json";

const originalState = rawData.map((d) => ({
  time: new Date(d.time),
  value: d.value,
}));

const states = ref([originalState]);

const saves = ref([{ label: "Original", time: "Initial" }]);

const activeIndex = ref(0);

function saveState(currentData) {
  states.value.push(
    currentData.map((d) => ({
      time: new Date(d.time),
      value: d.value,
    }))
  );

  saves.value.push({
    label: `Save ${saves.value.length}`,
    time: new Date().toLocaleTimeString(),
  });

  activeIndex.value = states.value.length - 1;
}

function selectState(index) {
  activeIndex.value = index;
}
</script>

<style>
#app {
  text-align: center;
  background-color: rgb(206, 215, 215);
  user-select: none;
}
</style>
