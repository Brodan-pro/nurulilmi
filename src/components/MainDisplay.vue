<script setup>
import { ref, onMounted, onUnmounted, watch, nextTick, computed } from "vue";

const props = defineProps({
  images: {
    type: Array,
    default: () => [],
  },
});

const currentIndex = ref(0);
const isTransitioning = ref(false);
let intervalId = null;

const SLIDE_INTERVAL = 8000;
const TRANSITION_MS = 1200;

// Duplikat gambar pertama di akhir agar seamless
const extendedImages = computed(() => {
  if (!props.images.length) return [];
  return [...props.images, props.images[0]];
});

// Translasi container
const translateX = computed(() => `translateX(-${currentIndex.value * 100}%)`);

// Preload semua gambar agar langsung tampil full
function preloadImages() {
  props.images.forEach((img) => {
    const image = new Image();
    image.src = img.URL_Cloudinary;
  });
}

// Jalankan slider infinite
function startSlider() {
  clearInterval(intervalId);
  if (props.images.length > 1) {
    intervalId = setInterval(async () => {
      isTransitioning.value = true;
      await nextTick();
      currentIndex.value += 1;

      // Setelah slide terakhir (duplikat pertama), reset ke awal tanpa animasi
      if (currentIndex.value === props.images.length) {
        setTimeout(() => {
          isTransitioning.value = false;
          currentIndex.value = 0;
        }, TRANSITION_MS);
      }
    }, SLIDE_INTERVAL);
  }
}

onMounted(() => {
  preloadImages();
  // 🔹 Tambahkan delay kecil biar layout TV fix dulu sebelum render pertama
  setTimeout(() => {
    startSlider();
  }, 500);
});

onUnmounted(() => clearInterval(intervalId));

// Jika spreadsheet update
watch(
  () => props.images.length,
  async (newLen, oldLen) => {
    if (newLen > oldLen) preloadImages();
    await nextTick();
    if (currentIndex.value >= newLen) currentIndex.value = 0;
    startSlider();
  }
);
</script>

<template>
  <div
    class="relative flex-1 overflow-hidden rounded-xl bg-black/20 backdrop-blur-sm border border-white/10 select-none"
  >
    <!-- Pastikan container punya tinggi fix agar tidak shrink -->
    <div
      class="absolute inset-0 flex w-full h-full"
      :class="{
        'transition-transform duration-[1200ms] ease-[cubic-bezier(0.45,0,0.55,1)]': isTransitioning,
      }"
      :style="{ transform: translateX }"
    >
      <div
        v-for="(img, i) in extendedImages"
        :key="i + '-' + img.URL_Cloudinary"
        class="flex-shrink-0 w-full h-full"
      >
        <img
          :src="img.URL_Cloudinary"
          :alt="img.Judul"
          class="w-full h-full object-cover object-center select-none"
          draggable="false"
        />
      </div>
    </div>

    <!-- Placeholder -->
    <div
      v-if="!props.images.length"
      class="absolute inset-0 flex items-center justify-center text-3xl text-white/50"
    >
      ZONA GAMBAR UTAMA
    </div>

    <!-- Overlay gelap lembut -->
    <div
      class="absolute inset-0 pointer-events-none bg-gradient-to-t from-black/25 via-transparent to-black/10"
    ></div>
  </div>
</template>

<style scoped>
/* Pastikan layout penuh dari frame pertama */
:host,
div,
img {
  width: 100%;
  height: 100%;
  max-height: 100%;
}

img {
  object-fit: cover;
  will-change: transform;
  backface-visibility: hidden;
  transform: translateZ(0);
}
</style>
