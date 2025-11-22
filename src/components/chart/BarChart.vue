<template>
  <Bar :data="chartData" :options="chartOptions" />
</template>
<!-- Sample only  -->
<script setup>
import { Bar } from 'vue-chartjs'
import {
  Chart,
  BarElement,
  CategoryScale,
  LinearScale,
  Tooltip,
  Legend
} from 'chart.js'

// Register chart components
Chart.register(BarElement, CategoryScale, LinearScale, Tooltip, Legend)

// Chart Data
const chartData = {
  labels: ['Red', 'Blue', 'Yellow', 'Green'],
  datasets: [
    {
      label: 'Votes',
      data: [12, 19, 3, 5],
      backgroundColor: ['red', 'blue', 'yellow', 'green']
    }
  ]
}

// Chart Options
const chartOptions = {
  responsive: true,
  maintainAspectRatio: false
}
</script>

<style>
/* Optional: control chart height */
canvas {
  max-height: 400px;
}
</style>
