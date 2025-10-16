<script setup>
console.log('🚀 FinancialReport.vue: Initialized');

defineProps({
  report: Array,
});
</script>

<template>
  <div class="p-4">
    <h3 class="text-xl font-bold text-[#000000] mb-3">Laporan Keuangan</h3>
    <div v-for="(item, index) in report" :key="index" class="flex justify-between text-sm py-1">
      <span>{{ item.keterangan }}</span>
      <span class="font-medium">{{ item.jumlah }}</span>
    </div>
  </div>
</template>