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
import { toast } from "vue-sonner";
import axios from "axios";
import getStudents, { fetchStudents } from "@/api/api";
import { Loader2 } from "lucide-vue-next";

Chart.register(BarElement, CategoryScale, LinearScale, Tooltip, Legend);

// ----------------------------------------------------
// Reactive variables
// ----------------------------------------------------
const chartData = ref(null);
const isLoading = ref(true)
const chartOptions = {
  responsive: true,
  maintainAspectRatio: false, // important for responsive height
  scales: {
    y: { 
      beginAtZero: true,
      ticks: { stepSize: 1 }
    }
  },
  plugins: {
    legend: {
      position: "top"
    },
    tooltip: {
      enabled: true
    }
  }
};

const sessionId = localStorage.getItem("session_id_utm_ttms");
const baseUrl = import.meta.env.VITE_BASE_URL;
const adminId = localStorage.getItem("admin_id_utm_ttms");
// Analyze amount of students per department

onMounted(async () => {
  try {
    const students = await fetchStudents(adminId, "2025/2026", 1);
  
    const counts = {};

    // Grouping logic
    students.forEach(s => {
      const key = s.kod_kursus;
      let category = 'Others';

      if(['SECJ', 'SCSJ', 'SECJH', 'SCSEH'].includes(key)) category = 'Software Engineering';
      else if (['SECR', 'SECRH'].includes(key)) category = 'Network Security';
      else if (key === 'SECVH') category = 'Graphic & Multimedia';
      else if (key === 'SECPH') category = 'Data Engineering';
      else if (key === 'SECBH') category = 'Bioinformatics';

      counts[category] = (counts[category] || 0) + 1;
    });

    // 1. Convert object to array: [['Software', 310], ['Network', 262], ...]
    // 2. Sort by the value (index 1) in descending order
    const sortedData = Object.entries(counts)
      .sort((a, b) => b[1] - a[1]);

    // 3. Extract sorted labels and values
    const labels = sortedData.map(entry => entry[0]);
    const values = sortedData.map(entry => entry[1]);

    chartData.value = {
      labels,
      datasets: [
        {
          label: "Total students",
          data: values,
          backgroundColor: [
            "rgba(75, 192, 192, 0.7)", // Largest
            "rgba(255, 99, 132, 0.7)",
            "rgba(255, 205, 86, 0.7)",
            "rgba(54, 162, 235, 0.7)",
            "rgba(153, 102, 255, 0.7)",
            "rgba(201, 203, 207, 0.7)",
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

<style>

</style>
