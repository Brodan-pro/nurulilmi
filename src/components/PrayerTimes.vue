<script setup>
defineProps({
  schedule: Object,
  nextPrayerName: String,
});

const prayerList = [
  { key: "imsak", label: "Imsak" },
  { key: "subuh", label: "Subuh" },
  { key: "terbit", label: "Terbit" },
  { key: "dzuhur", label: "Dzuhur" },
  { key: "ashar", label: "Ashar" },
  { key: "maghrib", label: "Maghrib" },
  { key: "isya", label: "Isya" },
];
</script>

<template>
  <aside
    class="flex flex-col justify-start h-full bg-black/10 p-4 rounded-xl border border-white/10 backdrop-blur-sm"
  >
    

    <div class="flex-1 grid grid-cols-1 gap-2">
      <div
        v-for="item in prayerList"
        :key="item.key"
        class="rounded-xl px-3 py-2 text-white transition-all duration-500 border border-white/10"
        :class="nextPrayerName === item.label
          ? 'bg-gradient-to-r from-[#F4A261] to-[#E76F51] shadow-lg scale-[1.03]'
          : 'bg-black/30 hover:bg-black/40'"
      >
        <div class="flex justify-between items-center">
          <span class="text-base font-semibold">{{ item.label }}</span>
          <span class="text-2xl font-black tracking-wider">
            {{ schedule[item.key] }}
          </span>
        </div>
      </div>
    </div>
  </aside>
</template>
