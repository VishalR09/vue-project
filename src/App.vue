<template>
  <div id="app">
    <h3>Line graph</h3>

    <Chart :data="items" @save="storeUpdatedData" />
  </div>
</template>

<script setup>
import { ref } from "vue";
import Chart from "./components/Chart.vue";
import rawData from "./server/chartData.json";

const items = ref(
  rawData.map((d) => ({
    time: new Date(d.time),
    value: d.value,
  }))
);

function storeUpdatedData(updatedData) {
  items.value = updatedData;
  console.log("Saved in App.vue:", updatedData);
}
</script>

<style>
#app {
  text-align: center;
  background-color: rgb(206, 215, 215);
  user-select: none;
}
</style>
