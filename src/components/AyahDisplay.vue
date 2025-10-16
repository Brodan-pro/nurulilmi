<script setup>
console.log('🚀 AyahDisplay.vue: Initialized');

defineProps({
  data: Object,
});
</script>

<template>
  <div class="w-full h-full flex flex-col items-center justify-center text-center p-4">
    <div class="text-5xl sm:text-6xl font-arabic text-white leading-loose mb-6">
      {{ data["Teks_Utama_Arab"] || data.arab || "Memuat Ayat..." }}
    </div>
    <div class="text-xl sm:text-2xl text-[#F4A261] font-medium max-w-full line-clamp-3">
      {{ data["Teks_Latin_Terjemahan"] || data.latin || data.indo || "Memuat terjemahan..." }}
    </div>
    <div class="mt-4 text-lg text-white/70">
      {{ data["Sumber_Referensi"] || "Sumber: ..." }}
    </div>
  </div>
</template>

<style scoped>
.line-clamp-3 {
  display: -webkit-box;
  -webkit-line-clamp: 3 ;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>