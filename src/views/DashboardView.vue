<template>
  <div class="flex-1 flex items-center justify-center p-8 md:pt-40 pt-50">
    <div class="flex-1 mx-auto grid gap-8">
      <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
        <div
          v-for="stat in stats"
          :key="stat.label"
          class="bg-white p-6 rounded-3xl shadow-sm border border-slate-100"
        >
          <p class="text-slate-500 text-sm font-medium">{{ stat.label }}</p>
          <h3 class="text-2xl font-bold mt-1">{{ stat.value }}</h3>
          <div :class="stat.trendColor" class="text-xs font-bold mt-2 flex items-center">
            {{ stat.trend }}
          </div>
        </div>
      </div>

      <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
        <div class="bg-white p-8 rounded-3xl shadow-sm border border-slate-100">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-lg font-bold text-slate-900">Sentiment Distribution</h3>
            <button
              v-if="selectedSentiment"
              @click="selectedSentiment = null"
              class="text-xs text-indigo-600 hover:underline font-bold cursor-pointer"
            >
              Show Overall
            </button>
          </div>
          <div class="space-y-4">
            <div
              v-for="item in sentimentData"
              :key="item.label"
              @click="selectedSentiment = item.label"
              class="cursor-pointer rounded-2xl p-3 transition-all"
              :class="[
                selectedSentiment === item.label
                  ? 'bg-indigo-50 shadow-inner'
                  : 'hover:bg-slate-50',
                selectedSentiment && selectedSentiment !== item.label ? 'opacity-40 scale-95' : '',
              ]"
            >
              <div class="flex justify-between text-sm pb-1.5">
                <span class="font-bold text-slate-700">{{ item.label }}</span>
                <span class="font-bold text-slate-900">{{ item.value }}%</span>
              </div>
              <div class="w-full bg-slate-100 h-1.5 rounded-full overflow-hidden">
                <div
                  class="h-full transition-all duration-700 rounded-full"
                  :class="item.barColor"
                  :style="{ width: item.value + '%' }"
                ></div>
              </div>
            </div>
          </div>
        </div>

        <div class="bg-white p-8 rounded-3xl shadow-sm border border-slate-100 flex flex-col">
          <div class="pb-6">
            <h3 class="text-lg font-bold text-slate-900">
              {{ selectedSentiment ? `${selectedSentiment} Insights` : 'Overall Highlights' }}
            </h3>
            <p class="text-xs text-slate-400 italic">Top keywords detected in your voice logs</p>
          </div>

          <div class="flex flex-col sm:flex-row items-center justify-center gap-8 w-full">
            <div class="relative w-48 h-48 mx-auto">
              <svg viewBox="0 0 36 36" class="w-full h-full transform -rotate-90">
                <circle
                  cx="18"
                  cy="18"
                  r="15.915"
                  fill="none"
                  class="stroke-slate-100"
                  stroke-width="4"
                />
                <circle
                  v-for="trigger in triggerPieData"
                  :key="trigger.label"
                  cx="18"
                  cy="18"
                  r="15.915"
                  fill="none"
                  :class="trigger.color"
                  stroke-width="4"
                  :stroke-dasharray="`${trigger.percentage} ${100 - trigger.percentage}`"
                  :stroke-dashoffset="trigger.offset"
                  class="transition-all duration-700 ease-in-out"
                />
              </svg>
              <div class="absolute inset-0 flex flex-col items-center justify-center text-center">
                <span class="text-3xl font-black text-slate-800">{{ currentTotalCount }}</span>
                <span class="text-[10px] font-bold text-slate-400 uppercase tracking-tighter"
                  >Occurrences</span
                >
              </div>
            </div>

            <div
              class="flex flex-row flex-wrap sm:flex-col gap-3 w-full sm:w-fit justify-center sm:justify-normal mx-auto"
            >
              <div
                v-for="trigger in triggerPieData"
                :key="trigger.label"
                class="flex items-center gap-3"
              >
                <div class="w-3 h-3 rounded-full" :class="trigger.color"></div>
                <div class="flex flex-col">
                  <span class="text-sm font-bold text-slate-700 leading-tight">{{
                    trigger.label
                  }}</span>
                  <span class="text-[10px] font-medium text-slate-400"
                    >{{ trigger.percentage }}% intensity</span
                  >
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="lg:col-span-2 bg-white p-8 rounded-3xl shadow-sm border border-slate-100">
          <h3 class="text-lg font-bold mb-6">Recent Journal Entries</h3>
          <div class="overflow-x-auto">
            <table class="w-full text-left min-w-[600px]">
              <thead>
                <tr class="text-slate-400 text-sm border-b border-slate-50">
                  <th class="pb-4 font-medium">Date & Time</th>
                  <th class="pb-4 font-medium">Topic Highlight</th>
                  <th class="pb-4 font-medium">Sentiment</th>
                  <th class="pb-4 font-medium text-right">Duration</th>
                </tr>
              </thead>
              <tbody class="text-sm">
                <tr
                  v-for="entry in paginatedHistory"
                  :key="entry.id"
                  class="border-b border-slate-50 last:border-0"
                >
                  <td class="py-4 text-slate-600 whitespace-nowrap">{{ entry.date }}</td>
                  <td class="py-4 font-medium text-slate-800">{{ entry.topic }}</td>
                  <td class="py-4">
                    <span
                      :class="entry.statusClass"
                      class="px-3 py-1 rounded-full text-[10px] font-bold uppercase tracking-wider"
                    >
                      {{ entry.sentiment }}
                    </span>
                  </td>
                  <td class="py-4 text-right text-slate-400">{{ entry.duration }}</td>
                </tr>
              </tbody>
            </table>
          </div>

          <div
            v-if="isExpanded && totalPages > 1"
            class="flex items-center justify-center gap-4 pt-8"
          >
            <button
              @click="currentPage--"
              :disabled="currentPage === 1"
              class="p-2 rounded-xl hover:bg-slate-100 disabled:opacity-30 disabled:pointer-none transition-colors flex items-center gap-4 cursor-pointer"
            >
              <IconNext class="rotate-180" />
            </button>

            <div class="flex gap-1">
              <button
                v-for="(page, index) in visiblePages"
                :key="index"
                @click="page !== '...' && (currentPage = page)"
                class="w-8 h-8 rounded-lg text-xs font-bold transition-all cursor-pointer"
                :class="[
                  page === currentPage
                    ? 'bg-black text-white shadow-md'
                    : 'text-slate-400 hover:bg-slate-100',
                  page === '...' ? 'cursor-default hover:bg-transparent' : '',
                ]"
                :disabled="page === '...'"
              >
                {{ page }}
              </button>
            </div>

            <button
              @click="currentPage++"
              :disabled="currentPage === totalPages"
              class="p-2 rounded-xl hover:bg-slate-100 disabled:opacity-30 disabled:pointer-none transition-colors flex items-center gap-4 cursor-pointer"
            >
              <IconNext />
            </button>
          </div>
          <div class="flex justify-center pt-6">
            <button
              @click="isExpanded = !isExpanded"
              class="flex flex-col items-center text-slate-400 hover:text-slate-600 transition cursor-pointer"
            >
              <IconNext
                class="transition-transform duration-300"
                :class="isExpanded ? 'rotate-90' : 'rotate-270'"
              />
              <span class="text-xs font-medium mt-1">
                {{ isExpanded ? 'Show Less' : 'Show More' }}
              </span>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import IconNext from '@/components/icons/IconNext.vue'
import { ref, computed, watch } from 'vue'

const selectedSentiment = ref(null)

const isExpanded = ref(false)

const currentPage = ref(1)
const itemsPerPage = 10

const paginatedHistory = computed(() => {
  if (!isExpanded.value) {
    return history.value.slice(0, 3)
  }

  const start = (currentPage.value - 1) * itemsPerPage
  const end = start + itemsPerPage
  return history.value.slice(start, end)
})

const totalPages = computed(() => Math.ceil(history.value.length / itemsPerPage))

const stats = ref([
  {
    label: 'Avg. Peace Score',
    value: '78/100',
    trend: '↑ 12% from last week',
    trendColor: 'text-green-600',
  },
  {
    label: 'Total Recordings',
    value: '24 Entries',
    trend: '6 this week',
    trendColor: 'text-indigo-600',
  },
  {
    label: 'Most Frequent Emotion',
    value: 'Positive',
    trend: 'Stable',
    trendColor: 'text-slate-500',
  },
])

const sentimentData = ref([
  { label: 'Positive', value: 45, barColor: 'bg-green-500' },
  { label: 'Neutral', value: 30, barColor: 'bg-slate-400' },
  { label: 'Negative', value: 15, barColor: 'bg-red-500' },
  { label: 'Mixed', value: 10, barColor: 'bg-orange-400' },
])

// ข้อมูล Insight ที่จะเปลี่ยนไปตามอารมณ์ที่เลือก
const insightsBySentiment = {
  Overall: [
    { label: 'Work/Job', count: 20, color: 'stroke-indigo-500 bg-indigo-500' },
    { label: 'Family', count: 10, color: 'stroke-sky-400 bg-sky-400' },
    { label: 'Health', count: 8, color: 'stroke-emerald-500 bg-emerald-500' },
    { label: 'Other', count: 7, color: 'stroke-slate-300 bg-slate-300' },
  ],
  Positive: [
    { label: 'Workout', count: 12, color: 'stroke-green-500 bg-green-500' },
    { label: 'Vacation', count: 8, color: 'stroke-teal-400 bg-teal-400' },
    { label: 'Promotion', count: 5, color: 'stroke-lime-500 bg-lime-500' },
    { label: 'New Hobby', count: 5, color: 'stroke-emerald-300 bg-emerald-300' },
  ],
  Negative: [
    { label: 'Deadline', count: 14, color: 'stroke-red-600 bg-red-600' },
    { label: 'Traffic', count: 10, color: 'stroke-rose-400 bg-rose-400' },
    { label: 'Budget', count: 6, color: 'stroke-orange-500 bg-orange-500' },
    { label: 'Conflict', count: 4, color: 'stroke-red-300 bg-red-300' },
  ],
  Neutral: [
    { label: 'Chores', count: 9, color: 'stroke-slate-400 bg-slate-400' },
    { label: 'Weather', count: 7, color: 'stroke-gray-300 bg-gray-300' },
    { label: 'Commute', count: 5, color: 'stroke-zinc-500 bg-zinc-500' },
  ],
  Mixed: [
    { label: 'Feedback', count: 6, color: 'stroke-amber-500 bg-amber-500' },
    { label: 'Meetings', count: 5, color: 'stroke-indigo-300 bg-indigo-300' },
    { label: 'Socializing', count: 4, color: 'stroke-violet-400 bg-violet-400' },
  ],
}

const currentInsightData = computed(() => {
  return insightsBySentiment[selectedSentiment.value] || insightsBySentiment.Overall
})

const currentTotalCount = computed(() => {
  return currentInsightData.value.reduce((sum, i) => sum + i.count, 0)
})

const triggerPieData = computed(() => {
  let currentOffset = 0
  return currentInsightData.value.map((item) => {
    const percentage = Math.round((item.count / currentTotalCount.value) * 100)
    const offset = currentOffset
    currentOffset -= percentage
    return { ...item, percentage, offset }
  })
})

const visiblePages = computed(() => {
  const total = totalPages.value
  const current = currentPage.value

  if (total <= 5) {
    return Array.from({ length: total }, (_, i) => i + 1)
  }

  if (current <= 3) {
    return [1, 2, 3, '...', total]
  }

  if (current >= total - 2) {
    return [1, '...', total - 2, total - 1, total]
  }

  return [1, '...', current - 1, current, current + 1, '...', total]
})

const history = ref([
  {
    id: 1,
    date: 'Oct 24, 2023 - 09:15 AM',
    topic: 'Stressing about the cloud project deadline',
    sentiment: 'Negative',
    statusClass: 'bg-red-100 text-red-600',
    duration: '2:45',
  },
  {
    id: 2,
    date: 'Oct 23, 2023 - 08:30 PM',
    topic: 'Relaxing evening after gym session',
    sentiment: 'Positive',
    statusClass: 'bg-green-100 text-green-600',
    duration: '1:20',
  },
  {
    id: 3,
    date: 'Oct 22, 2023 - 01:10 PM',
    topic: 'Lunch meeting with the design team',
    sentiment: 'Neutral',
    statusClass: 'bg-slate-100 text-slate-600',
    duration: '3:05',
  },
  {
    id: 4,
    date: 'Oct 24, 2023 - 09:15 AM',
    topic: 'Stressing about the cloud project deadline',
    sentiment: 'Negative',
    statusClass: 'bg-red-100 text-red-600',
    duration: '2:45',
  },
  {
    id: 5,
    date: 'Oct 23, 2023 - 08:30 PM',
    topic: 'Relaxing evening after gym session',
    sentiment: 'Positive',
    statusClass: 'bg-green-100 text-green-600',
    duration: '1:20',
  },
  {
    id: 6,
    date: 'Oct 22, 2023 - 01:10 PM',
    topic: 'Lunch meeting with the design team',
    sentiment: 'Neutral',
    statusClass: 'bg-slate-100 text-slate-600',
    duration: '3:05',
  },
  {
    id: 7,
    date: 'Oct 24, 2023 - 09:15 AM',
    topic: 'Stressing about the cloud project deadline',
    sentiment: 'Negative',
    statusClass: 'bg-red-100 text-red-600',
    duration: '2:45',
  },
  {
    id: 8,
    date: 'Oct 23, 2023 - 08:30 PM',
    topic: 'Relaxing evening after gym session',
    sentiment: 'Positive',
    statusClass: 'bg-green-100 text-green-600',
    duration: '1:20',
  },
  {
    id: 9,
    date: 'Oct 22, 2023 - 01:10 PM',
    topic: 'Lunch meeting with the design team',
    sentiment: 'Neutral',
    statusClass: 'bg-slate-100 text-slate-600',
    duration: '3:05',
  },
  {
    id: 10,
    date: 'Oct 24, 2023 - 09:15 AM',
    topic: 'Stressing about the cloud project deadline',
    sentiment: 'Negative',
    statusClass: 'bg-red-100 text-red-600',
    duration: '2:45',
  },
  {
    id: 11,
    date: 'Oct 23, 2023 - 08:30 PM',
    topic: 'Relaxing evening after gym session',
    sentiment: 'Positive',
    statusClass: 'bg-green-100 text-green-600',
    duration: '1:20',
  },
  {
    id: 12,
    date: 'Oct 22, 2023 - 01:10 PM',
    topic: 'Lunch meeting with the design team',
    sentiment: 'Neutral',
    statusClass: 'bg-slate-100 text-slate-600',
    duration: '3:05',
  },
])

watch(isExpanded, (val) => {
  if (!val) {
    currentPage.value = 1
  }
})
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

circle {
  transition:
    stroke-dasharray 0.7s ease,
    stroke-dashoffset 0.7s ease;
}
</style>
