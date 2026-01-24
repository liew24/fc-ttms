<script setup>
    import CoursesStudent from '@/components/chart/students/CoursesStudent.vue';
    import DepartmentStudent from '@/components/chart/students/DepartmentStudent.vue';
    import StudentList from '@/components/chart/students/StudentList.vue';
    import YearStudents from '@/components/chart/students/YearStudents.vue';
</script>
<template>
  <div class="max-w-7xl mx-auto p-4 md:p-8 space-y-8">
    
     <header class="text-center md:text-left border-b pb-4">
      <h1 class="text-2xl font-bold text-gray-800">Student Analysis Dashboard</h1>
      <p class="text-sm text-gray-500">Academic Year: 2025/2026 | Session 1</p>
    </header>


    <div class="flex flex-col space-y-8">
      
      <section class="w-full bg-white shadow-sm border border-gray-100 rounded-2xl p-6">
        <h2 class="font-bold text-lg mb-4 flex items-center gap-2">
          <span class="w-2 h-6 bg-indigo-600 rounded-full"></span>
          Student List
        </h2>
        <div class="overflow-x-auto">
          <StudentList />
        </div>
      </section>

      <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
        
        <article class="flex flex-col bg-white shadow-sm border border-gray-100 rounded-2xl p-5 hover:shadow-md transition-shadow relative min-h-[400px]">
          <h3 class="font-semibold text-gray-700 mb-2">Subject Workload</h3>
          <p class="text-xs text-gray-500 mb-4">Amount of subjects taken by student</p>
          
          <div v-if="isLoading" class="absolute inset-0 flex items-center justify-center bg-white bg-opacity-80 rounded-2xl z-10">
            <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-indigo-600"></div>
          </div>

          <div class="flex-grow flex items-center justify-center">
             <CoursesStudent />
          </div>
        </article>

        <article class="flex flex-col bg-white shadow-sm border border-gray-100 rounded-2xl p-5 hover:shadow-md transition-shadow relative min-h-[400px]">
          <h3 class="font-semibold text-gray-700 mb-2">Department Enrollment</h3>
          <p class="text-xs text-gray-500 mb-4">Amount of students per department</p>
          
          <div v-if="isLoading" class="absolute inset-0 flex items-center justify-center bg-white bg-opacity-80 rounded-2xl z-10">
            <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-indigo-600"></div>
          </div>

          <div class="flex-grow flex items-center justify-center">
            <DepartmentStudent />
          </div>
        </article>

        <article class="lg:col-span-2 flex flex-col bg-white shadow-sm border border-gray-100 rounded-2xl p-5 hover:shadow-md transition-shadow relative min-h-[400px]">
          <div class="text-center mb-4">
            <h3 class="font-semibold text-gray-700">Year of Study Distribution</h3>
            <p class="text-xs text-gray-500">Student spread across academic years</p>
          </div>

          <div v-if="isLoading" class="absolute inset-0 flex items-center justify-center bg-white bg-opacity-80 rounded-2xl z-10">
            <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-indigo-600"></div>
          </div>

          <div class="flex-grow flex items-center justify-center">
            <YearStudents />
          </div>
        </article>

      </div>
    </div>
  </div>
</template>


<style lang="scss" scoped>

</style>