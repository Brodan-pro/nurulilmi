<script setup>
defineProps({
  text: String,
});
</script>

<template>
  <footer
    class="flex-none min-h-[60px] w-full 
           bg-gradient-to-b from-gray-50 to-white
           flex items-center 
           border-t border-gray-200 z-10"
  >
    <p
      v-if="text"
      class="running-text text-lg md:text-xl text-slate-800 px-4 select-none drop-shadow-sm" >
      {{ text }}
    </p>
    <p v-else class="px-4 text-slate-500 select-none">Tidak ada pesan.</p>
  </footer>
</template>

<style scoped>
.running-text {
  display: inline-block;
  padding-left: 100%;
  white-space: nowrap;
  animation: marquee 40s linear infinite;
}

@keyframes marquee {
  0%   { transform: translateX(0); }
  100% { transform: translateX(-100%); }
}
</style>
