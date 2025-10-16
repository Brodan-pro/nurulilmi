<script setup>
defineProps({
  data: Object,
});
</script>

<template>
  <div class="w-full h-full flex flex-col items-center justify-center text-center p-4">
    <div class="text-3xl sm:text-4xl font-sans text-white leading-relaxed mb-6">
      {{ data["Teks_Latin_Terjemahan"] || data.latin || data.indo || "Memuat Hadits..." }}
    </div>
    <div class="mt-4 text-lg text-white/70">
      {{ data["Sumber_Referensi"] || "Sumber: ..." }}
    </div>
  </div>
</template>

<style scoped>
.line-clamp-4 {
  display: -webkit-box;
  -webkit-line-clamp: 4;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>