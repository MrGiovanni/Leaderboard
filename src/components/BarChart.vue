<template>
  <div class="chart-wrapper">
    <Bar :data="chartData" :options="chartOptions" />
  </div>
</template>

<script>
import { Bar } from "vue-chartjs";
import {
  Chart,
  BarElement,
  CategoryScale,
  LinearScale,
  Tooltip,
  Legend,
} from "chart.js";

// Register the Chart.js components
Chart.register(BarElement, CategoryScale, LinearScale, Tooltip, Legend);

export default {
  name: "BarChart",
  components: {
    Bar,
  },
  props: {
    chartData: {
      type: Object,
      required: true,
    },
    chartOptions: {
      type: Object,
      default: () => ({}),
    },
  },
};
</script>

<style scoped>
.chart-wrapper {
  position: relative;
  height: 300px;
}
</style>
