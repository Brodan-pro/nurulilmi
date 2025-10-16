<script setup>
defineProps({
  data: Object,
});
</script>

<template>
  <div class="w-full h-full flex flex-col items-center justify-center text-center p-4">
    <!-- Doa Arab -->
    <div class="text-4xl sm:text-5xl font-arabic text-white leading-relaxed mb-6">
      {{ data["Teks_Utama_Arab"] || data.arab || "Memuat Doa..." }}
    </div>
    <!-- Terjemahan -->
    <div class="text-xl sm:text-2xl text-[#F4A261] font-medium max-w-full line-clamp-4">
      {{ data["Teks_Latin_Terjemahan"] || data.latin || data.indo || "Memuat terjemahan..." }}
    </div>
    <!-- Sumber -->
    <div class="mt-4 text-lg text-white/70">
      {{ data["Sumber_Referensi"] || "Sumber: ..." }}
    </div>
  </div>
</template>

<style scoped>
/* Gunakan Tailwind line-clamp, tidak perlu CSS tambahan */
</style>