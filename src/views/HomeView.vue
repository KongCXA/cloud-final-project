<template>
  <main
    class="flex-1 flex flex-col items-center justify-center p-6 relative overflow-hidden h-screen bg-white"
  >
    <div
      class="transition-all duration-500"
      :class="{ 'pr-80': isSidebarOpen }"
    >
      <VoiceRecord />
    </div>

    <button
      @click="toggleSidebar"
      class="fixed right-6 bottom-6 w-20 h-20 bg-black shadow-xl rounded-full flex items-center justify-center text-white z-50 border cursor-pointer hover:scale-105 transition-transform"
    >
      <span>{{ isSidebarOpen ? "close" : "history" }}</span>
    </button>

    <Transition name="slide">
      <div
        v-if="isSidebarOpen"
        class="absolute right-0 top-0 h-full w-80 bg-white shadow-2xl border-l border-slate-100 z-40 flex flex-col"
      >
        <div
          class="p-6 border-b border-slate-50 flex justify-between items-center"
        >
          <h2 class="font-bold tracking-tight text-slate-800">
            Recent Reflections
          </h2>
          <span
            class="text-[10px] bg-slate-200 px-2 py-0.5 rounded font-bold uppercase"
            >History</span
          >
        </div>

        <div class="flex-1 overflow-y-auto p-4 flex flex-col gap-4">
          <div
            v-for="item in recentTranscripts"
            :key="item.id"
            @click="toggleExpand(item.id)"
            class="bg-white p-4 rounded-2xl border border-slate-100 shadow-sm hover:border-indigo-300 transition-all cursor-pointer group"
          >
            <div class="flex justify-between items-center mb-2">
              <span class="text-[10px] font-medium text-slate-400">{{
                item.time
              }}</span>
              <span
                :class="getSentimentStyle(item.sentiment)"
                class="text-[9px] px-2 py-0.5 rounded-full font-bold uppercase tracking-tighter border"
              >
                {{ item.sentiment }}
              </span>
            </div>

            <p
              class="text-xs leading-relaxed italic text-slate-600 transition-all duration-300"
              :class="{ 'line-clamp-2': expandedId !== item.id }"
            >
              "{{ item.text }}"
            </p>

            <div class="mt-2 flex justify-end">
              <span
                class="text-[9px] text-indigo-400 font-bold uppercase opacity-0 group-hover:opacity-100 transition-opacity"
              >
                {{
                  expandedId === item.id ? "Show Less" : "Click to Read More"
                }}
              </span>
            </div>
          </div>

          <div v-if="loading" class="text-center py-10">
            <p class="text-xs animate-pulse text-slate-400">
              Fetching your peace...
            </p>
          </div>
          <div
            v-else-if="recentTranscripts.length === 0"
            class="text-center py-10"
          >
            <p class="text-xs italic text-slate-400">No reflections yet...</p>
          </div>
        </div>
      </div>
    </Transition>

    <div
      v-if="isSidebarOpen"
      @click="isSidebarOpen = false"
      class="fixed inset-0 bg-slate-900/10 z-30 lg:hidden"
    ></div>
  </main>
</template>

<script setup lang="ts">
import { ref } from "vue";
import VoiceRecord from "@/components/VoiceRecord.vue";

const isSidebarOpen = ref(false);
const loading = ref(false);
const expandedId = ref<string | null>(null); // OpenSearch ID มักเป็น String
const recentTranscripts = ref<any[]>([]);

const getSentimentStyle = (sentiment: string) => {
  const s = sentiment?.toUpperCase();
  if (s === "POSITIVE") return "border-green-100 bg-green-50 text-green-600";
  if (s === "NEGATIVE") return "border-red-100 bg-red-50 text-red-600";
  return "border-slate-100 bg-slate-50 text-slate-600";
};

// ฟังก์ชันดึงข้อมูลจาก API Gateway
const fetchHistory = async () => {
  const apiUrl = import.meta.env.VITE_API_HISTORY_URL;
  if (!apiUrl) return;

  loading.value = true;
  try {
    const response = await fetch(apiUrl);
    const data = await response.json();

    // แปลงรูปแบบข้อมูลจาก OpenSearch (hits.hits) ให้เข้ากับหน้าเว็บ
    // สมมติว่า Lambda คืนค่ามาเป็น list ของ _source
    recentTranscripts.value = data.map((item: any) => ({
      id: item._id || Math.random().toString(),
      time: new Date(item._source.timestamp).toLocaleString("en-US", {
        hour: "numeric",
        minute: "numeric",
        hour12: true,
        day: "numeric",
        month: "short",
      }),
      text: item._source.transcript,
      sentiment: item._source.sentiment || "NEUTRAL",
    }));
  } catch (error) {
    console.error("Error fetching history:", error);
  } finally {
    loading.value = false;
  }
};

const toggleSidebar = () => {
  isSidebarOpen.value = !isSidebarOpen.value;
  if (isSidebarOpen.value) {
    fetchHistory();
  }
};

const toggleExpand = (id: string) => {
  expandedId.value = expandedId.value === id ? null : id;
};
</script>

<style scoped>
/* ใช้ Tailwind line-clamp (มีมาให้ใน Tailwind v3+) */
.line-clamp-2 {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
}

/* Animation อื่นๆ คงเดิม */
.slide-enter-active,
.slide-leave-active {
  transition: transform 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.slide-enter-from,
.slide-leave-to {
  transform: translateX(100%);
}

.overflow-y-auto::-webkit-scrollbar {
  width: 4px;
}
.overflow-y-auto::-webkit-scrollbar-thumb {
  background: #e2e8f0;
  border-radius: 10px;
}
</style>
