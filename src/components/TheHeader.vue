<script setup>
import logoUlbi from '../assets/logos/logoulbi.jpg';
import logoYpbpi from '../assets/logos/logoypbpi.jpg';
import logodkm from '../assets/logos/logodkm.jpg';

defineProps({
  masehiDate: String,
  hijriDateStr: String,
});
</script>

<template>
  <header
    class="flex items-center justify-between px-6 py-3 
           bg-gradient-to-b from-gray-50 to-white 
           border-b border-gray-200
           min-h-[70px] sm:min-h-[80px] md:min-h-[90px]
           flex-shrink-0 w-full z-10"
  >
    <div class="flex items-center gap-2 sm:gap-4 h-full"> 
      <div class="bg-white p-1 rounded-md shadow-md flex items-center justify-center h-12">
        <img 
          :src="logoYpbpi" 
          alt="Logo YPBPI" 
          class="h-8 w-auto object-contain" />
      </div>

      <div class="flex items-center gap-2 sm:gap-4">
        <div class="bg-white p-1 rounded-md shadow-md flex items-center justify-center h-12">
          <img 
            :src="logoUlbi" 
            alt="Logo ULBI" 
            class="h-10 w-auto object-contain" />
        </div>
        <div class="bg-white p-1 rounded-md shadow-md flex items-center justify-center h-12">
          <img 
            :src="logodkm" 
            alt="Logo DKM" 
            class="h-10 w-auto object-contain" />
        </div>
      </div>
    </div>
    
    <div class="flex-1 text-center">
      <h1 class="text-xl sm:text-2xl md:text-3xl font-bold tracking-wider text-slate-800 drop-shadow-md whitespace-nowrap">
        MASJID NURUL ILMI
      </h1>
    </div>

    <div class="text-right leading-tight">
      <div v-if="masehiDate && hijriDateStr">
        <p class="text-sm md:text-base font-bold text-slate-700">{{ masehiDate }}</p>
        <p class="text-xs md:text-sm text-[#B8860B] font-semibold">{{ hijriDateStr }}</p>
      </div>
      <div v-else class="text-sm text-slate-500">Memuat tanggal...</div>
    </div>
  </header>
</template>