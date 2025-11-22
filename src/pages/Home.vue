<script setup>
import BarChart from "@/components/chart/BarChart.vue";
import { Button } from "@/components/ui/button"; // shadcn Button import
import { useUserStore } from "@/stores/user";
import { ChartLine, GraduationCap, House, LibraryBig, School, Sheet, Users, UserStar } from "lucide-vue-next";
import { ref } from "vue";
import { useRouter } from "vue-router";
import { toast } from "vue-sonner";

const drawerOpen = ref(false)
const user = useUserStore()
const router = useRouter()

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

</script>
<template>
    <div>
        <main class="p-6">
            <h1 class="text-2xl font-bold mb-4">Welcome Back {{ user && user.name }}!</h1>
            <p class="text-gray-700">
                This is a simple homepage with a navbar and a logout button using shadcn components.
                <BarChart/>
            </p>
        </main>
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
