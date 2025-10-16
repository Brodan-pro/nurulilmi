<script setup>
defineProps({ data: Object });
</script>

<template>
  <div class="flex flex-col justify-between items-center h-full py-8 px-4 text-center">
    <p class="text-2xl text-slate-700 font-semibold mb-2">
      Asmaul Husna
    </p>

    <div class="flex-1 flex flex-col justify-center items-center w-full">
      <h3
        class="text-7xl font-bold font-arabic leading-snug bg-gradient-to-b from-gray-700 to-black text-transparent bg-clip-text"
        style="line-height: 1.3;"
      >
        {{ data?.arab }}
      </h3>

      <h4
        class="text-3xl font-medium mt-4 bg-gradient-to-b from-gray-700 to-black text-transparent bg-clip-text"
        style="line-height: 1.4;"
      >
        {{ data?.latin }}
      </h4>

      <p
        class="text-lg text-slate-600 italic mt-3 max-w-[90%]"
        style="line-height: 1.6;"
      >
        “{{ data?.indo }}”
      </p>
    </div>

    <div class="h-4"></div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Scheherazade+New:wght@400;700&display=swap');

.font-arabic {
  font-family: 'Scheherazade New', serif;
  font-weight: 700;
  direction: rtl;
  text-align: center;
  word-break: keep-all;
}
</style>
