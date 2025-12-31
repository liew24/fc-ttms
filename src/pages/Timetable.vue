<script setup>
import { ref, onMounted, computed } from "vue";
import {
    Search, Filter, Eye, ArrowLeft, Loader2, ArrowUpDown, X, MapPin, ChevronLeft, ChevronRight, ChevronDown, ChevronsUp
} from "lucide-vue-next";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Card, CardContent } from "@/components/ui/card";
import axios from "axios"; 
import { useUserStore } from "@/stores/user";

// --- STATE MANAGEMENT ---
const userStore = useUserStore();
const selectedItem = ref(null);
const searchQuery = ref("");
const loading = ref(false);
const error = ref("");
const timetableData = ref([]);

// SESSION STATE
const currentSesi = ref("");
const currentSem = ref("");

const expandedDays = ref({});

// --- SORTING & FILTERING STATE ---
const sortBy = ref('time'); 
const isFilterOpen = ref(false); 
const selectedDay = ref(null);   

// --- MAPPING ---
const availableDays = [
    { value: 2, label: "Monday", color: "bg-red-50 border-red-100 text-black-700", badge: "bg-red-200 text-black-800" },
    { value: 3, label: "Tuesday", color: "bg-orange-50 border-red-100 text-black-700", badge: "bg-orange-200 text-black-800" },
    { value: 4, label: "Wednesday", color: "bg-green-50 border-red-100 text-black-700", badge: "bg-green-200 text-black-800" },
    { value: 5, label: "Thursday", color: "bg-blue-50 border-red-100 text-black-700", badge: "bg-blue-200 text-black-800" },
    { value: 6, label: "Friday", color: "bg-purple-50 border-red-100 text-black-700", badge: "bg-purple-200 text-black-800" }, 
];

// --- COMPUTED PROPERTIES ---
const filteredTimetable = computed(() => {
    let data = [...timetableData.value];

    if (searchQuery.value) {
        const query = searchQuery.value.toLowerCase();
        data = data.filter(item => {
            const nameMatch = item.nama_subjek.toLowerCase().includes(query);
            const codeMatch = item.kod_subjek.toLowerCase().includes(query);
            return nameMatch || codeMatch;
        });
    }
    return data;
});

// --- HELPERS ---
const toggleDay = (dayValue) => {
    expandedDays.value[dayValue] = !expandedDays.value[dayValue];
}

const getClassesForDay = (dayValue) => {
    return filteredTimetable.value.filter(item => item.hari == dayValue);
}

const formatDay = (day) => {
    const days = { 1: "Sunday", 2: "Monday", 3: "Tuesday", 4: "Wednesday", 5: "Thursday", 6: "Friday", 7: "Saturday" };
    return days[day] || day; 
}

const formatTime = (startMasa, endMasa) => {
    const end = endMasa || startMasa; 

    const startHour = parseInt(startMasa) + 6; 
    const endHour = parseInt(end) + 6; 

    const pad = (n) => n < 10 ? '0' + n : n;
    
    return `${pad(startHour)}00 - ${pad(endHour)}50`;
}

// --- API ACTIONS ---
const fetchTimetable = async () => {
    loading.value = true;
    error.value = "";
    if (userStore.sessionToken === "") {
        error.value = "Session lost. Please Logout and Login again.";
        loading.value = false;
        return;
    }

    try {
        let allSubjects = [];
        
        try {
            const studentRes = await axios.get('http://web.fc.utm.my/ttms/web_man_webservice_json.cgi', {
                params: { entity: 'pelajar_subjek', no_matrik: localStorage.getItem('matric_no')  }
            });
            console.log("studentRes.data:", studentRes.data, "matric_no:", userStore.matric_no);
            if (Array.isArray(studentRes.data) && studentRes.data.length > 0) {
                allSubjects = studentRes.data;
            }
        } catch (e) {
            console.log(e.message);
        }
        if (allSubjects.length === 0) {
            try {
                const lecturerRes = await axios.get('http://web.fc.utm.my/ttms/web_man_webservice_json.cgi', {
                    params: { entity: 'pensyarah_subjek', no_pekerja: localStorage.getItem('matric_no') }
                });
                if (Array.isArray(lecturerRes.data) && lecturerRes.data.length > 0) {
                    allSubjects = lecturerRes.data;
                }
            } catch (e) {
                console.log(e.message);
            }
        }

        if (allSubjects.length === 0) {
            error.value = "No subjects found for this semester.";
            loading.value = false;
            return;
        }

        currentSesi.value = allSubjects[0].sesi;     
        currentSem.value = allSubjects[0].semester;  
        
        const currentSubjects = allSubjects.filter(sub => 
            sub.sesi === currentSesi.value && sub.semester == currentSem.value
        );

        const detailedRequests = currentSubjects.map(async (subject) => {
            
            const schedulePromise = await axios.get('http://web.fc.utm.my/ttms/web_man_webservice_json.cgi', {
                params: {
                    entity: 'jadual_subjek',
                    sesi: currentSesi.value,
                    semester: currentSem.value,
                    kod_subjek: subject.kod_subjek,
                    seksyen: subject.seksyen
                }
            });

            const lecturerPromise = await axios.get('http://web.fc.utm.my/ttms/web_man_webservice_json.cgi', {
                params: {
                    entity: 'subjek_pensyarah',
                    sesi: currentSesi.value,
                    semester: currentSem.value,
                    kod_subjek: subject.kod_subjek,
                    seksyen: subject.seksyen
                }
            });

            const [scheduleRes, lecturerRes] = await Promise.allSettled([schedulePromise, lecturerPromise]);

            let rawSchedules = [];
            if (scheduleRes.status === 'fulfilled' && Array.isArray(scheduleRes.value.data)) {
                rawSchedules = scheduleRes.value.data;
            }

            const schedules = rawSchedules.filter(s => s && s.masa);
            
            let lecturerName = "Not Assigned";
            if (lecturerRes.status === 'fulfilled' && lecturerRes.value.data && lecturerRes.value.data.length > 0) {
                const lect = lecturerRes.value.data[0];
                lecturerName = lect.nama || lect.nama_pensyarah || lect.nama_staf || "Unknown";
            }

            if (schedules.length === 0) {
                return [{
                    ...subject,
                    hari: "TBA", masa: 0, ruang: null, lecturer_name: lecturerName
                }];
            }

            return schedules.map(slot => ({
                ...subject,       
                ...slot,
                masa: parseInt(slot.masa),
                lecturer_name: lecturerName 
            }));
        });

        const nestedResults = await Promise.all(detailedRequests);
        
        const sortedData = nestedResults.flat().sort((a, b) => {
            if (a.hari != b.hari) return a.hari - b.hari;
            return a.masa - b.masa;
        });

        const mergedData = [];
        if (sortedData.length > 0) {
            let currentBlock = { ...sortedData[0], endMasa: sortedData[0].masa };

            for (let i = 1; i < sortedData.length; i++) {
                const nextSlot = sortedData[i];

                const isSameDay = nextSlot.hari == currentBlock.hari;
                const isSameSubject = nextSlot.kod_subjek == currentBlock.kod_subjek;
                const isSameSection = nextSlot.seksyen == currentBlock.seksyen;
                const isConsecutive = parseInt(nextSlot.masa) === parseInt(currentBlock.endMasa) + 1;

                if (isSameDay && isSameSubject && isSameSection && isConsecutive) {
                    currentBlock.endMasa = nextSlot.masa;
                } else {
                    mergedData.push(currentBlock);
                    currentBlock = { ...nextSlot, endMasa: nextSlot.masa };
                }
            }
            mergedData.push(currentBlock);
        }

        timetableData.value = mergedData;

    } catch (err) {
        console.error(err);
        error.value = "Error loadging timetable. Please try again later.";
    } finally {
        loading.value = false;
    }
};

onMounted(() => {
    fetchTimetable();
});

const viewDetails = (item) => {
    selectedItem.value = item;
}
const goBack = () => {
    selectedItem.value = null;
}

const toggleFilter = () => { isFilterOpen.value = !isFilterOpen.value; }
const selectDay = (dayValue) => { selectedDay.value = dayValue; isFilterOpen.value = false; }
const clearFilter = () => { selectedDay.value = null; isFilterOpen.value = false; }
</script>

<template>
    <div class="p-4 md:p-6 w-full min-h-screen" @click="isFilterOpen = false">
        
        <div v-if="loading" class="flex flex-col items-center justify-center h-64 text-gray-500">
            <Loader2 class="w-8 h-8 animate-spin mb-2 text-primary" />
            <p>Loading your timetable...</p>
        </div>

        <div v-else-if="error" class="bg-red-50 text-red-600 p-4 rounded-md border border-red-200 text-center">
            {{ error }}
        </div>

        <div v-else-if="!selectedItem">
            
            <div class="mb-6">
                 <h1 class="text-2xl font-bold text-primary">Timetable</h1>
                 <p class="text-gray-500 text-sm" v-if="timetableData.length > 0">
                    Session {{ currentSesi || '...' }} |     {{ currentSem || '...' }}
                 </p>
            </div>

            <div class="bg-purple-50/50 p-4 rounded-lg mb-6 flex items-center relative z-10">
                <div class="relative w-full">
                    <Search class="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400 w-4 h-4" />
                    <Input v-model="searchQuery" placeholder="Search Subject Name or Code" class="pl-10 bg-white rounded-full border-none shadow-sm" />
                </div>
            </div>

            <div class="space-y-3">
                <div v-for="day in availableDays" :key="day.value">
                    
                    <div class="border border-gray-100 rounded-lg overflow-hidden bg-white shadow-sm transition-all hover:shadow-md">
                        
                        <div 
                            @click="toggleDay(day.value)" 
                            class="p-4 flex justify-between items-center cursor-pointer select-none transition-colors"
                            :class="[
                                day.color, // Apply color ALWAYS
                                expandedDays[day.value] ? 'border-b' : '' // Add a separator line only when open
                            ]"
                        >
                            <div class="flex items-center gap-3">
                                <div class="w-6 h-6 rounded-full flex items-center justify-center text-xs font-bold"
                                     :class="day.badge"> {{ getClassesForDay(day.value).length }}
                                </div>
                                <span class="font-bold uppercase tracking-wide">
                                    {{ day.label }}
                                </span>
                            </div>
                            
                            <component :is="expandedDays[day.value] ? ChevronUp : ChevronDown" 
                                       class="w-5 h-5 opacity-70" />
                        </div>

                        <div v-if="expandedDays[day.value]" class="bg-white">
                            
                            <div v-if="getClassesForDay(day.value).length > 0">
                                <div v-for="(item, i) in getClassesForDay(day.value)" :key="i" 
                                     class="flex items-center justify-between p-4 border-b border-gray-50 last:border-0 hover:bg-purple-50/30 transition-colors pl-6 md:pl-12">
                                    
                                    <div class="w-2/3 pr-2">
                                        <p class="text-sm font-bold text-gray-800">{{ item.nama_subjek }}</p>
                                        <p class="text-xs text-gray-500 uppercase mt-1">
                                            {{ item.kod_subjek }} <span class="text-gray-300">|</span> Sec {{ item.seksyen }}
                                        </p>
                                    </div>

                                    <div class="flex items-center gap-3">
                                        <span class="text-[10px] md:text-xs font-mono text-primary bg-primary/5 px-2 py-1 rounded whitespace-nowrap">
                                            {{ formatTime(item.masa, item.endMasa) }}
                                        </span>
                                        <button @click.stop="viewDetails(item)" class="text-gray-400 hover:text-primary transition-colors">
                                            <Eye class="w-5 h-5" />
                                        </button>
                                    </div>
                                </div>
                            </div>

                            <div v-else class="p-6 text-center text-gray-400 text-xs italic">
                                No classes scheduled for {{ day.label }}.
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            
        </div>

        <div v-else>
            <button @click="goBack" class="flex items-center gap-2 text-primary font-medium mb-6 hover:underline">
                <ArrowLeft class="w-5 h-5" />
                Back
            </button>

            <Card class="w-full shadow-md border border-gray-100">
                <CardContent class="p-6 space-y-8 relative">
                    
                    <div>
                        <h2 class="text-xl font-bold text-primary mb-1 uppercase">
                            {{ selectedItem.nama_subjek }}
                        </h2>
                        <p class="text-sm text-gray-500">
                            {{ selectedItem.kod_subjek }}
                        </p>
                    </div>

                    <div class="grid grid-cols-1 md:grid-cols-2 gap-y-8 gap-x-4">
                        <div>
                            <p class="text-xs text-gray-400 font-medium mb-1">Schedule</p>
                            <p class="text-sm font-medium text-gray-800">
                                {{ formatDay(selectedItem.hari) }}, {{ formatTime(selectedItem.masa, selectedItem.endMasa) }}
                            </p>
                        </div>

                        <div>
                            <p class="text-xs text-gray-400 font-medium mb-1">Location</p>
                            <div v-if="selectedItem.ruang">
                                <p class="text-sm font-bold text-gray-800">{{ selectedItem.ruang.kod_ruang }}</p> 
                                <p class="text-xs text-gray-500 uppercase">{{ selectedItem.ruang.nama_ruang }}</p>
                            </div>
                            <div v-else>
                                <p class="text-sm font-medium text-gray-800">Online / TBA</p>
                            </div>
                        </div>

                        <div>
                            <p class="text-xs text-gray-400 font-medium mb-1">Lecturer</p>
                            <div class="flex items-center gap-2">
                                <div class="bg-gray-100 p-1 rounded-full">
                                    <svg class="w-4 h-4 text-gray-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"></path></svg>
                                </div>
                                <p class="text-sm font-medium text-gray-800 uppercase">
                                    {{ selectedItem.lecturer_name }}
                                </p>
                            </div>
                        </div>

                        <div>
                            <p class="text-xs text-gray-400 font-medium mb-1">Section</p>
                            <p class="text-sm font-medium text-gray-800">{{ selectedItem.seksyen }}</p>
                        </div>
                    </div>
                </CardContent>
            </Card>
        </div>

    </div>
</template>