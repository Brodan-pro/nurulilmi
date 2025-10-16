<script setup>
import { computed } from 'vue';

const props = defineProps({
  prayerName: String,
  duration: Number,
  elapsed: Number,
});

const progressPercentage = computed(() => {
  return props.duration > 0 ? (props.elapsed / props.duration) * 100 : 0;
});
</script>

<template>
  <div class="fixed inset-0 bg-gradient-to-br from-gray-50 to-white flex flex-col items-center justify-center z-50 p-8">
    
    <h2 class="text-6xl font-bold mb-8 bg-gradient-to-b from-gray-700 to-black text-transparent bg-clip-text">
      Sedang Melaksanakan Sholat {{ prayerName }}
    </h2>
    
    <p class="text-3xl text-slate-600 mb-16">
      Mohon menjaga ketenangan.
    </p>
    
    <div class="w-full max-w-4xl bg-gray-200 rounded-full h-4 overflow-hidden">
      <div 
        class="bg-gradient-to-r from-amber-400 to-amber-600 h-4 rounded-full transition-all duration-1000 ease-linear" 
        :style="{ width: progressPercentage + '%' }"
      ></div>
    </div>
  </div>
</template>