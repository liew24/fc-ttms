<script setup>
import LectureList from '@/components/chart/lecturers/LecturerList.vue';
import LecturerSection from '@/components/chart/lecturers/LecturerSection.vue';
import LecturerStudents from '@/components/chart/lecturers/LecturerStudents.vue';
import LecturerSubject from '@/components/chart/lecturers/LecturerSubject.vue';

   
</script>
<template>
    <!-- Lee Hom -->
     <!-- Can only be access by admin id -->
    <div class="w-full h-full flex items-center justify-center my-10 space-y-6 flex-col">

        <div class=" flex flex-col items-center justify-center space-y-4">
            <!-- TODO: User can customize the data based on semester, year -->
            <p class="font-bold ">Lecturer List</p>
            <LectureList/>
        </div>

        <!-- TODO: Add loading animation -->
        <div class=" flex flex-col items-center justify-center space-y-2 shadow-md rounded-2xl px-3 py-2">
            <!-- TODO: User can customize the data based on semester, year -->
            <p class="font-bold text-sm md:text-md ">Distribution of subjects taught</p>
            <LecturerSubject/>
        </div>

        <div class=" flex flex-col items-center justify-center space-y-2 shadow-md rounded-2xl px-3 py-2">
            <!-- TODO: User can customize the data based on semester, year -->
            <p class="font-bold text-sm md:text-md">Lecturer workload by number of students</p>
            <!-- <p class="text-xs md:text-sm ">Shows distribution of students accross lecturers.</p> -->
            <LecturerStudents/>
        </div>
        <div class=" flex flex-col items-center justify-center space-y-2 shadow-md rounded-2xl px-3 py-2">
            <!-- TODO: User can customize the data based on semester, year -->
            <p class="font-bold text-sm md:text-md">Lecturer workload by number of section</p>
            <!-- <p class="text-xs md:text-sm">Shows distribution of sections accross lecturers.</p> -->
            <LecturerSection/>
        </div>
        
    </div>
</template>


<style lang="scss" scoped>

</style>