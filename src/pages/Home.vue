<script setup>
import BarChart from "@/components/chart/students/CoursesStudent.vue";
import { Button } from "@/components/ui/button"; // shadcn Button import
import { useUserStore } from "@/stores/user";
import { ChartLine, GraduationCap, House, LibraryBig, Loader2, School, Sheet, Users, UserStar } from "lucide-vue-next";
import { onMounted, ref } from "vue";
import { useRouter } from "vue-router";
import { toast } from "vue-sonner";
import { Bar } from "vue-chartjs";
import {
    Chart as ChartJS,
    Title,
    Tooltip,
    Legend,
    BarElement,
    CategoryScale,
    LinearScale,
} from "chart.js";
import axios from "axios";

ChartJS.register(CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend);

const students = ref([])
// students info
const studentSession = ref('')
const studentYear = ref(0)
const studentSem = ref('')

const drawerOpen = ref(false)
const user = useUserStore()
const router = useRouter()
const admissionInfo = ref({ session: "Loading...", semester: "-", year: "-" });
const semesters = ref([]);
const chartData = ref({ labels: [], datasets: [] });
const chartOptions = ref({ responsive: true });
const isLoading = ref(true)

function toggleDrawer() {
    drawerOpen.value = !drawerOpen.value
}
const handleLogout = () => {
    // Add your logout logic here, e.g., clearing auth tokens
    console.log("user logging out")
    user.logout()
    toast.success("Logged out successfully!", { id: "logout-success" })
    router.push("login")
};

const goToPages = (page) => {
    router.push(page)
}

const loadCurriculumRoadmap = async () => {
    const mockResponse = {
        session: "2023/2024",
        current_semester: "1",
        current_year: "2",
        roadmap: [
            {
                sem_id: 1,
                name: "Year 1 - Sem 1",
                total_credit: 15,
                subjects: [
                    { code: "SECJ1013", name: "Programming Technique I", type: "Core", credit: 3 },
                    { code: "SECI1013", name: "Discrete Structure", type: "Core", credit: 3 },
                    { code: "UHIS1022", name: "Philosophy & Current Issues", type: "Elective", credit: 2 },
                    { code: "SECJ1013", name: "Technology & Info System", type: "Core", credit: 3 },
                ]
            },
            {
                sem_id: 2,
                name: "Year 1 - Sem 2",
                total_credit: 12,
                subjects: [
                    { code: "SECJ1023", name: "Programming Technique II", type: "Core", credit: 3 },
                    { code: "SECP1013", name: "Database Systems", type: "Core", credit: 3 },
                    { code: "UHLB1112", name: "English Comm Skills", type: "University", credit: 2 },
                ]
            }
        ]
    };
    admissionInfo.value = {
        session: mockResponse.session,
        semester: mockResponse.current_semester,
        year: mockResponse.current_year
    };
    semesters.value = mockResponse.roadmap;

    chartData.value = {
        labels: mockResponse.roadmap.map(s => s.name),
        datasets: [{
            label: 'Total Credit Hours',
            backgroundColor: '#800000',
            data: mockResponse.roadmap.map(s => s.total_credit)
        }]
    };
};

onMounted(() => {
    loadCurriculumRoadmap();
});

onMounted(async () => {
    try {
        const response = await axios.get(import.meta.env.VITE_BASE_URL, {
            params: {
                'entity': 'pelajar_subjek',
                'no_matrik': user.matric_no
            }
        })
        // console.log(`response data: ${JSON.stringify(response.data, null, 2)}`)
        students.value = response.data

        if (students.value.length > 0) {
            console.log('First student object:', students.value[0].sesi);
        }
        studentSession.value = students.value[0].sesi
        studentYear.value = students.value[0].tahun_kursus
        studentSem.value = students.value[0].semester
        console.log(`isloading? ${isLoading.value}`)
    } catch (error) {
        console.log("Error fetching students list: ", error);
        toast.error("Failed to load students list.", {
            id: "students-load-failed",
        });
    } finally {
        isLoading.value = false;
    }
});

</script>

<template>
    <div v-if="isLoading" class="flex items-center justify-center h-screen">
        <Loader2 class="animate-spin text-primary h-8 w-8" />
    </div>
    <div class="p-6 space-y-6" v-if="!isLoading">
        <h1 class="text-2xl font-bold text-primary">Welcome back {{ user.name }}</h1>

        <div class="bg-white p-4 rounded shadow border-l-4 border-primary">
            <h2 class="text-lg font-semibold">Admission Info</h2>
            <div class="flex gap-6 mt-2 text-gray-700">
                <div><span class="font-bold" v-if="studentSession !== ''">Session:</span> {{ studentSession }}</div>
                <div><span class="font-bold" v-if="studentYear !== 0">Current Year:</span> {{ studentYear }}</div>
                <div><span class="font-bold" v-if="studentSem != 0">Semester:</span> {{ studentSem }}</div>
            </div>
        </div>

        <div class="bg-white p-4 rounded shadow">
            <h2 class="text-lg font-semibold mb-4">Credit Hours Distribution</h2>
            <div class="h-64">
                <Bar v-if="chartData.labels.length" :data="chartData" :options="chartOptions" />
            </div>
        </div>

        <div class="grid md:grid-cols-2 gap-6">
            <div v-for="sem in semesters" :key="sem.sem_id" class="bg-white border rounded-lg shadow-sm p-4">
                <div class="flex justify-between border-b pb-2 mb-2">
                    <h3 class="font-bold text-primary">{{ sem.name }}</h3>
                    <span class="text-xs font-semibold bg-gray-200 px-2 py-1 rounded">Total Credits: {{ sem.total_credit
                        }}</span>
                </div>
                <ul class="space-y-3">
                    <li v-for="sub in sem.subjects" :key="sub.code" class="flex justify-between items-start text-sm">
                        <div>
                            <div class="font-medium">{{ sub.code }}</div>
                            <div class="text-gray-500 text-xs">{{ sub.name }}</div>
                        </div>
                        <div class="text-right">
                            <span class="block font-bold">{{ sub.credit }} Credit</span>
                            <span
                                :class="{ 'text-red-600': sub.type === 'Core', 'text-blue-600': sub.type === 'Elective' }"
                                class="text-xs">
                                {{ sub.type }}
                            </span>
                        </div>
                    </li>
                </ul>
            </div>
        </div>
    </div>
</template>


<style scoped>
/* optional: make navbar sticky */
nav {
    position: sticky;
    top: 0;
    z-index: 50;
}
</style>
