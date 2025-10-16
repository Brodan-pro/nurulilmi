<script setup>
import AsmaulHusnaDisplay from "./AsmaulHusnaDisplay.vue";
import FinancialReport from "./FinancialReport.vue";

defineProps({
  activeContent: Object,
  financialReport: Array
});
</script>

<template>
  <aside
    class="h-full flex flex-col justify-between bg-white border border-gray-200 rounded-2xl shadow-lg overflow-hidden"
  >
    <section
      class="flex-1 p-4 flex flex-col items-center justify-center text-center relative"
    >
      <Transition name="fade" mode="out-in">
        <div
          v-if="activeContent && activeContent.type === 'asmaulHusna'"
          :key="activeContent.data?.arab ?? 'asmaul-husna'"
          class="w-full h-full flex flex-col items-center justify-center"
        >
          
          <AsmaulHusnaDisplay :data="activeContent.data" />
        </div>
        <div
          v-else
          key="loading"
          class="flex items-center justify-center h-full text-2xl text-slate-500" >
          Memuat Asmaul Husna...
        </div>
      </Transition>
    </section>

    <section
      class="flex-shrink-0 bg-gray-100 p-4 border-t border-gray-200"
    >
      
      <FinancialReport :report="financialReport" />
    </section>
  </aside>
</template>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease-in-out;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* Biar tampilan smooth di TV */
aside {
  height: 100%;
  min-height: 0;
  display: flex;
  flex-direction: column;
}
</style>
