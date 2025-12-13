<template>

  <div class="relative w-full max-w-full md:max-w-2xl md:w-200 md:h-80 h-50 mx-auto px-4">

    <!-- Loading Spinner -->
    <div v-if="isLoading" class="absolute inset-0 flex items-center justify-center bg-white/60 z-20">
      <Loader2 class="w-10 h-10 animate-spin text-gray-600" />
    </div>

    <!-- Chart -->
    <div :class="{ 'opacity-30 pointer-events-none': isLoading }" class="w-full h-full">
      <Doughnut v-if="chartData" :data="chartData" :options="chartOptions" />
    </div>

  </div>

</template>


<script setup>
import { ref, onMounted } from "vue";
import { Doughnut } from "vue-chartjs";
import { Chart, ArcElement, Tooltip, Legend } from "chart.js";
import getStudents from "@/api/api";
import { Loader2 } from "lucide-vue-next";

Chart.register(ArcElement, Tooltip, Legend);

// ----------------------------------------------------
// Reactive variables
// ----------------------------------------------------
const chartData = ref(null);
const isLoading = ref(true);

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: { position: "top" },
    tooltip: { enabled: true }
  }
};

const sessionId = localStorage.getItem("session_id_utm_ttms");

// Load chart data
onMounted(async () => {
  try {
    const students = await getStudents("pensyarah", sessionId, "2024/2025", 1, 0);

    // Count by number of subjects
    const countBySubjects = {};

    students.forEach(s => {
      const key = s.bil_subjek;
      if (!countBySubjects[key]) countBySubjects[key] = 0;
      countBySubjects[key]++;
    });

    const labels = Object.keys(countBySubjects);
    const values = Object.values(countBySubjects);

    chartData.value = {
      labels,
      datasets: [
        {
          label: "Amount of subjects taken by lecturers",
          data: values,
          backgroundColor: [
            "rgba(255, 99, 132, 0.7)",
            "rgba(54, 162, 235, 0.7)",
            "rgba(255, 206, 86, 0.7)",
            "rgba(75, 192, 192, 0.7)",
            "rgba(153, 102, 255, 0.7)",
            "rgba(255, 159, 64, 0.7)"
          ]
        }
      ]
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