<script setup>
import LectureList from '@/components/chart/lecturers/LecturerList.vue';
import LecturerSection from '@/components/chart/lecturers/LecturerSection.vue';
import LecturerStudents from '@/components/chart/lecturers/LecturerStudents.vue';
import LecturerSubject from '@/components/chart/lecturers/LecturerSubject.vue';

   
</script>
<template>
  <div class="max-w-7xl mx-auto p-4 md:p-8 space-y-8">
    
    <header class="text-center md:text-left border-b pb-4">
      <h1 class="text-2xl font-bold text-gray-800">Lecturer Analysis Dashboard</h1>
      <p class="text-sm text-gray-500">Academic Year: 2025/2026 | Session 1</p>
    </header>

    <div class="flex flex-col space-y-8">
      
      <section class="w-full bg-white shadow-sm border border-gray-100 rounded-2xl p-6">
        <h2 class="font-bold text-lg mb-4 flex items-center gap-2">
          <span class="w-2 h-6 bg-blue-600 rounded-full"></span>
          Lecturer List
        </h2>
        <div class="overflow-x-auto">
          <LectureList />
        </div>
      </section>

      <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
        
        <article class="flex flex-col bg-white shadow-sm border border-gray-100 rounded-2xl p-5 hover:shadow-md transition-shadow">
          <h3 class="font-semibold text-gray-700 mb-4">Subject Distribution</h3>
          <div class="h-64 md:h-80 w-full flex items-center justify-center">
             <LecturerSubject />
          </div>
        </article>

        <article class="flex flex-col bg-white shadow-sm border border-gray-100 rounded-2xl p-5 hover:shadow-md transition-shadow">
          <h3 class="font-semibold text-gray-700 mb-4">Workload (Student Count)</h3>
          <div class="h-64 md:h-80 w-full flex items-center justify-center">
            <LecturerStudents />
          </div>
        </article>

        <article class="lg:col-span-2 flex flex-col bg-white shadow-sm border border-gray-100 rounded-2xl p-5 hover:shadow-md transition-shadow">
          <h3 class="font-semibold text-gray-700 mb-4 text-center">Workload (Section Count)</h3>
          <div class="h-64 md:h-80 w-full flex items-center justify-center">
            <LecturerSection />
          </div>
        </article>

      </div>
    </div>
  </div>
</template>


<style lang="scss" scoped>

</style>