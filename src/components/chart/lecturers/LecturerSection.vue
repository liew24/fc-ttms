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
    // 1. Rename variable for clarity (it fetches lecturers, not students)
    const lecturers = await getStudents("pensyarah", sessionId, "2025/2026", 1, 0);

    const countBySection = {};

    lecturers.forEach(lecturer => {
      // default to 0 if null/undefined to avoid crashes
      const sectionCount = lecturer.bil_seksyen || 0; 
      
      if (!countBySection[sectionCount]) countBySection[sectionCount] = 0;
      countBySection[sectionCount]++;
    });

    // 2. CRITICAL: Sort the keys numerically (1, 2, 3...) 
    // Otherwise "10" might appear before "2"
    const sortedSectionCounts = Object.keys(countBySection)
        .sort((a, b) => Number(a) - Number(b));

    // 3. Create descriptive labels
    const labels = sortedSectionCounts.map(count => `${count} Sections`);
    const values = sortedSectionCounts.map(count => countBySection[count]);

    chartData.value = {
      labels,
      datasets: [
        {
          label: "Number of Lecturers",
          data: values,
          // 4. Use a single color (or gradient) for distribution charts
          // Random colors (rainbow) can be distracting for this data type
          backgroundColor: "rgba(75, 192, 192, 0.6)",
          borderColor: "rgba(75, 192, 192, 1)",
          borderWidth: 1,
          barPercentage: 0.6, // Makes bars thinner and more elegant
        }
      ]
    };

    // 5. Add Axis Titles for Context
    chartOptions.value = {
      responsive: true,
      plugins: {
        legend: { display: false }, // Hide legend (title explains enough)
        title: {
          display: true,
          text: 'Lecturer Workload Distribution (By Sections)'
        },
        tooltip: {
            callbacks: {
                label: (context) => `${context.raw} Lecturers have this workload`
            }
        }
      },
      scales: {
        y: {
          beginAtZero: true,
          title: { display: true, text: 'Number of Lecturers' },
          ticks: { stepSize: 1 } // Avoids decimals like "1.5 lecturers"
        },
        x: {
          title: { display: true, text: 'Workload Size' }
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