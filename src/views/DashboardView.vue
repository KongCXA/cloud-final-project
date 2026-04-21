<template>
  <div
    class="flex-1 flex items-center justify-center p-8 md:pt-40 pt-50 bg-white"
  >
    <div class="flex-1 mx-auto grid gap-8">
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-8">
        <div
          v-for="stat in stats"
          :key="stat.label"
          class="bg-white p-6 rounded-3xl shadow-sm border border-slate-100"
        >
          <p class="text-slate-500 text-sm font-medium">{{ stat.label }}</p>
          <h3 class="text-2xl font-bold mt-1 text-black">
            {{ stat.value }}
          </h3>
          <p
            v-if="stat.subLabel"
            class="text-[10px] font-bold text-slate-400 mt-1 uppercase"
          >
            {{ stat.subLabel }}
          </p>
        </div>
      </div>

      <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
        <div class="bg-white p-8 rounded-3xl shadow-sm border border-slate-100">
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-lg font-bold text-slate-900">
              Sentiment Distribution
            </h3>
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
                selectedSentiment && selectedSentiment !== item.label
                  ? 'opacity-40 scale-95'
                  : '',
              ]"
            >
              <div class="flex justify-between text-sm pb-1.5">
                <span class="font-bold text-slate-700">{{ item.label }}</span>
                <span class="font-bold text-slate-900">{{ item.value }}%</span>
              </div>
              <div
                class="w-full bg-slate-100 h-1.5 rounded-full overflow-hidden"
              >
                <div
                  class="h-full transition-all duration-700 rounded-full"
                  :class="item.barColor"
                  :style="{ width: item.value + '%' }"
                ></div>
              </div>
            </div>
          </div>
        </div>

        <div
          class="bg-white p-8 rounded-3xl shadow-sm border border-slate-100 flex flex-col"
        >
          <div class="pb-6">
            <h3 class="text-lg font-bold text-slate-900">
              Sentiment Intensity
            </h3>
            <p class="text-xs text-slate-400 italic">
              Ratio between core emotions
            </p>
          </div>

          <div
            class="flex flex-col sm:flex-row items-center justify-center gap-8 w-full"
          >
            <div class="relative w-48 h-48 mx-auto">
              <svg
                viewBox="0 0 36 36"
                class="w-full h-full transform -rotate-90"
              >
                <circle
                  cx="18"
                  cy="18"
                  r="15.915"
                  fill="none"
                  class="stroke-slate-100"
                  stroke-width="4"
                />
                <circle
                  v-for="segment in triggerPieData"
                  :key="segment.label"
                  cx="18"
                  cy="18"
                  r="15.915"
                  fill="none"
                  :class="segment.color"
                  stroke-width="4"
                  :stroke-dasharray="`${segment.percentage} ${100 - segment.percentage}`"
                  :stroke-dashoffset="segment.offset"
                  class="transition-all duration-700 ease-in-out"
                />
              </svg>
              <div
                class="absolute inset-0 flex flex-col items-center justify-center text-center"
              >
                <span class="text-3xl font-black text-slate-800">
                  {{
                    selectedSentiment ? filteredHistory.length : history.length
                  }}
                </span>
                <span
                  class="text-[10px] font-bold text-slate-400 uppercase tracking-tighter"
                  >Total Logs</span
                >
              </div>
            </div>

            <div
              class="flex flex-row flex-wrap sm:flex-col gap-3 w-full sm:w-fit justify-center sm:justify-normal mx-auto"
            >
              <div
                v-for="segment in triggerPieData"
                :key="segment.label"
                class="flex items-center gap-3"
              >
                <div
                  class="w-3 h-3 rounded-full"
                  :class="segment.bgClass"
                ></div>
                <div class="flex flex-col">
                  <span
                    class="text-sm font-bold text-slate-700 leading-tight"
                    >{{ segment.label }}</span
                  >
                  <span class="text-[10px] font-medium text-slate-400"
                    >{{ segment.percentage }}% of total</span
                  >
                </div>
              </div>
            </div>
          </div>
        </div>

        <div
          class="lg:col-span-2 bg-white p-8 rounded-3xl shadow-sm border border-slate-100"
        >
          <h3 class="text-lg font-bold mb-6 text-black">
            Recent Journal Entries
          </h3>
          <div class="overflow-x-auto">
            <table class="w-full text-left min-w-[600px]">
              <thead>
                <tr class="text-slate-400 text-sm border-b border-slate-50">
                  <th class="pb-4 font-medium w-[200px]">Date & Time</th>
                  <th class="pb-4 font-medium">Journal Entry</th>
                  <th class="pb-4 pl-4 font-medium w-[120px]">Sentiment</th>
                </tr>
              </thead>
              <tbody class="text-sm">
                <tr
                  v-for="entry in paginatedHistory"
                  :key="entry.id"
                  class="border-b border-slate-50 last:border-0"
                >
                  <td class="py-4 text-slate-600 whitespace-nowrap">
                    {{ entry.date }}
                  </td>
                  <td class="py-4 font-medium text-slate-800">
                    <div
                      @click="toggleExpand(entry.id)"
                      class="cursor-pointer"
                      :class="[
                        expandedId === entry.id
                          ? 'whitespace-normal break-words max-w-full'
                          : 'max-w-[150px] md:max-w-[300px] lg:max-w-[500px] truncate',
                      ]"
                    >
                      {{ entry.topic }}
                    </div>
                  </td>
                  <td class="py-4 pl-4">
                    <span
                      :class="entry.statusClass"
                      class="px-3 py-1 rounded-full text-[10px] font-bold uppercase tracking-wider"
                    >
                      {{ entry.sentiment }}
                    </span>
                  </td>
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
                {{ isExpanded ? "Show Less" : "Show More" }}
              </span>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import IconNext from "@/components/icons/IconNext.vue";
import { ref, computed, watch, onMounted } from "vue";

const selectedSentiment = ref(null);
const isExpanded = ref(false);
const expandedId = ref("");
const loading = ref(false);
const currentPage = ref(1);
const itemsPerPage = 10;

const history = ref([]);

const fetchDashboardData = async () => {
  const apiUrl = import.meta.env.VITE_API_HISTORY_URL;
  if (!apiUrl) return;

  loading.value = true;
  try {
    const response = await fetch(apiUrl);
    const data = await response.json();

    history.value = data.map((item) => ({
      id: item._id,
      date: new Date(item._source.timestamp).toLocaleString("en-US", {
        month: "short",
        day: "numeric",
        year: "numeric",
        hour: "numeric",
        minute: "numeric",
        hour12: true,
      }),
      topic: item._source.transcript,
      sentiment: item._source.sentiment || "NEUTRAL",
      statusClass: getStatusClass(item._source.sentiment),
    }));
  } catch (error) {
    console.error("Error fetching dashboard data:", error);
  } finally {
    loading.value = false;
  }
};

const getStatusClass = (sentiment) => {
  const s = sentiment?.toUpperCase();
  if (s === "POSITIVE") return "bg-green-100 text-green-600";
  if (s === "NEGATIVE") return "bg-red-100 text-red-600";
  if (s === "MIXED") return "bg-orange-100 text-orange-600";
  return "bg-slate-100 text-slate-600";
};

const sentimentData = computed(() => {
  if (history.value.length === 0)
    return [
      { label: "Positive", value: 0, barColor: "bg-green-500" },
      { label: "Neutral", value: 0, barColor: "bg-slate-400" },
      { label: "Negative", value: 0, barColor: "bg-red-500" },
      { label: "Mixed", value: 0, barColor: "bg-orange-400" },
    ];

  const total = history.value.length;
  const counts = { POSITIVE: 0, NEGATIVE: 0, NEUTRAL: 0, MIXED: 0 };

  history.value.forEach((item) => {
    const s = item.sentiment.toUpperCase();
    if (counts.hasOwnProperty(s)) counts[s]++;
  });

  return [
    {
      label: "Positive",
      value: Math.round((counts.POSITIVE / total) * 100),
      barColor: "bg-green-500",
    },
    {
      label: "Neutral",
      value: Math.round((counts.NEUTRAL / total) * 100),
      barColor: "bg-slate-400",
    },
    {
      label: "Negative",
      value: Math.round((counts.NEGATIVE / total) * 100),
      barColor: "bg-red-500",
    },
    {
      label: "Mixed",
      value: Math.round((counts.MIXED / total) * 100),
      barColor: "bg-orange-400",
    },
  ];
});

const stats = computed(() => {
  const maxValue = Math.max(...sentimentData.value.map((s) => s.value));

  const topSentiments = sentimentData.value
    .filter((s) => s.value === maxValue && s.value > 0)
    .map((s) => s.label);

  let frequentEmotionLabel = "N/A";
  if (topSentiments.length > 0) {
    frequentEmotionLabel = topSentiments.join(" & ");
  }

  return [
    {
      label: "Total Recordings",
      value: `${history.value.length} Entries`,
      trendColor: "text-indigo-600",
    },
    {
      label: "Most Frequent Emotion",
      subLabel: topSentiments.length > 1 ? "Multiple Detected" : "Stable",
      value: history.value.length > 0 ? frequentEmotionLabel : "N/A",
      trendColor: "text-slate-500",
    },
  ];
});

const triggerPieData = computed(() => {
  const currentSentiments = sentimentData.value;
  const totalPercentage = currentSentiments.reduce(
    (sum, item) => sum + item.value,
    0,
  );

  let currentOffset = 0;
  return currentSentiments.map((item) => {
    const percentage =
      totalPercentage > 0
        ? Math.round((item.value / totalPercentage) * 100)
        : 0;
    const offset = currentOffset;
    currentOffset -= percentage;

    let strokeColor = "stroke-slate-400";
    if (item.label === "Positive") strokeColor = "stroke-green-500";
    if (item.label === "Negative") strokeColor = "stroke-red-500";
    if (item.label === "Mixed") strokeColor = "stroke-orange-400";

    return {
      label: item.label,
      percentage,
      offset,
      color: strokeColor,
      bgClass: item.barColor,
    };
  });
});

const filteredHistory = computed(() => {
  if (!selectedSentiment.value) return history.value;
  return history.value.filter(
    (item) =>
      item.sentiment.toUpperCase() === selectedSentiment.value.toUpperCase(),
  );
});

const paginatedHistory = computed(() => {
  const sourceData = filteredHistory.value;
  if (!isExpanded.value) return sourceData.slice(0, 3);
  const start = (currentPage.value - 1) * itemsPerPage;
  return sourceData.slice(start, start + itemsPerPage);
});

const totalPages = computed(() =>
  Math.ceil(filteredHistory.value.length / itemsPerPage),
);

const visiblePages = computed(() => {
  const total = totalPages.value;
  const current = currentPage.value;
  if (total <= 5) return Array.from({ length: total }, (_, i) => i + 1);
  if (current <= 3) return [1, 2, 3, "...", total];
  if (current >= total - 2) return [1, "...", total - 2, total - 1, total];
  return [1, "...", current - 1, current, current + 1, "...", total];
});

const toggleExpand = (id) => {
  expandedId.value = expandedId.value === id ? null : id;
};

onMounted(() => {
  fetchDashboardData();
});

watch(isExpanded, (val) => {
  if (!val) currentPage.value = 1;
});

watch(selectedSentiment, () => {
  currentPage.value = 1;
});
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
