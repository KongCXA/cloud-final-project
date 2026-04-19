<template>
  <main
    class="flex-1 flex flex-col items-center justify-center p-6 relative overflow-hidden h-screen"
  >
    <div class="transition-all duration-500" :class="{ 'pr-80': isSidebarOpen }">
      <VoiceRecord />
    </div>

    <button
      @click="isSidebarOpen = !isSidebarOpen"
      class="fixed right-6 bottom-6 w-20 h-20 bg-black shadow-xl rounded-full flex items-center justify-center text-white z-50 border cursor-pointer hover:scale-105 transition-transform"
    >
      <span>{{ isSidebarOpen ? 'close' : 'history' }}</span>
    </button>

    <Transition name="slide">
      <div
        v-if="isSidebarOpen"
        class="absolute right-0 top-0 h-full w-80 bg-white shadow-2xl border-l border-slate-100 z-40 flex flex-col"
      >
        <div class="p-6 border-b border-slate-50 flex justify-between items-center">
          <h2 class="font-bold tracking-tight text-slate-800">Recent Reflections</h2>
          <span class="text-[10px] bg-slate-200 px-2 py-0.5 rounded font-bold uppercase"
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
              <span class="text-[10px] font-medium text-slate-400">{{ item.time }}</span>
              <span
                :class="item.color"
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
                {{ expandedId === item.id ? 'Show Less' : 'Click to Read More' }}
              </span>
            </div>
          </div>

          <div v-if="recentTranscripts.length === 0" class="text-center py-10">
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
import { ref } from 'vue'
import VoiceRecord from '@/components/VoiceRecord.vue'

const isSidebarOpen = ref(false)
const expandedId = ref<number | null>(null) // เก็บ ID ของกล่องที่กำลังเปิด

const toggleExpand = (id: number) => {
  // ถ้ากดตัวเดิมให้ปิด ถ้ากดตัวใหม่ให้เปิดตัวใหม่
  expandedId.value = expandedId.value === id ? null : id
}

const recentTranscripts = ref([
  {
    id: 1,
    time: '10:30 AM',
    text: "I'm feeling a bit overwhelmed with work today, but I'll try to focus on one thing at a time. The cloud project is moving forward, but there are so many components to manage like S3, Transcribe, and Lambda. I need to remember that peace starts with a single breath.",
    sentiment: 'Negative',
    color: 'border-red-100 bg-red-50 text-red-600',
  },
  {
    id: 2,
    time: 'Yesterday',
    text: 'Had a great morning exercise. It really helped me start the day with a clear mind. Running in the park during sunrise is the best therapy.',
    sentiment: 'Positive',
    color: 'border-green-100 bg-green-50 text-green-600',
  },
  {
    id: 3,
    time: '2 days ago',
    text: 'Normal day at the office. Nothing much happened, just finished some routine tasks.',
    sentiment: 'Neutral',
    color: 'border-slate-100 bg-slate-50 text-slate-600',
  },
])
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
