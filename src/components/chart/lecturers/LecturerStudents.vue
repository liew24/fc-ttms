<template>

  <div class="relative w-full max-w-full md:max-w-2xl md:w-200 md:h-80 h-50 mx-auto px-4">

    <!-- Loading Spinner -->
    <div v-if="isLoading" class="absolute inset-0 flex items-center justify-center bg-white/60 z-20">
      <Loader2 class="w-10 h-10 animate-spin text-gray-600" />
    </div>

    <!-- Chart -->
    <div :class="{ 'opacity-30 pointer-events-none': isLoading }" class="w-full h-full">
      <Bar v-if="chartData" :data="chartData" :options="chartOptions" />
    </div>

  </div>

</template>

<script setup>
import { ref, onMounted } from "vue";
import { Bar } from "vue-chartjs";
import { Chart, BarElement, CategoryScale, LinearScale, Tooltip, Legend } from "chart.js";
import getStudents from "@/api/api";
import { Loader2 } from "lucide-vue-next";

Chart.register(BarElement, CategoryScale, LinearScale, Tooltip, Legend);

// ----------------------------------------------------
// Reactive variables
// ----------------------------------------------------
const chartData = ref(null);
const isLoading = ref(true);

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  scales: {
    x: {
      display: false, // This hides the entire X-axis (labels, line, and grid)
      grid: {
        display: false // Optional: explicitly ensures the vertical grid lines are gone
      }
    },
    y: { beginAtZero: true, ticks: { stepSize: 1 } }
  },
  plugins: {
    legend: { position: "top" },
    tooltip: { enabled: true }
  }
};

const sessionId = localStorage.getItem("session_id_utm_ttms");

// Load chart data
onMounted(async () => {
  try {
    const lecturers = await getStudents("pensyarah", sessionId, "2025/2026", 1, 0);

    // 1. SORT DATA: Ascending order (Smallest to Largest)
    // If you want Descending (Largest first), swap to: b.bil_pelajar - a.bil_pelajar
    lecturers.sort((a, b) => a.bil_pelajar - b.bil_pelajar);

    // 2. Prepare Labels and Data (now they follow the sorted order)
    const labels = lecturers.map(s => s.nama || 'Unknown'); 
    const values = lecturers.map(s => s.bil_pelajar);

    chartData.value = {
      labels,
      datasets: [
        {
          label: "Total Students",
          data: values,
          backgroundColor: [
            "#FF6384", "#36A2EB", "#FFCE56", "#4BC0C0", 
            "#9966FF", "#FF9F40", "#2ecc71", "#e74c3c"
          ],
          hoverOffset: 10
        }
      ]
    };

    chartOptions.value = {
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        legend: {
          display: true,
          position: 'right',
          labels: {
            padding: 20,
            usePointStyle: true,
          }
        },
        tooltip: {
          callbacks: {
            label: (context) => {
              const total = context.dataset.data.reduce((a, b) => a + b, 0);
              const value = context.raw;
              const percentage = ((value / total) * 100).toFixed(1);
              return `${context.label}: ${value} students (${percentage}%)`;
            }
          }
        }
      }
    };

  } catch (error) {
    console.error("Error loading chart:", error);
  } finally {
    isLoading.value = false;
  }
});
</script>

<style lang="scss" scoped>

</style>