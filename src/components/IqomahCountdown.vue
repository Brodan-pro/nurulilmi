<script setup>
import { computed, ref, onMounted, onUnmounted } from 'vue';

const props = defineProps({
  countdown: Number,
  prayerName: String,
});

const audioTick = ref(null);
const timerProgress = ref(100); // persen progress lingkaran
const initialCountdown = ref(props.countdown || 60);
const currentCountdown = ref(props.countdown || 60);
let countdownInterval = null;

// === RADIUS & LINGKARAN ===
const radius = 180; // radius px
const circumference = 2 * Math.PI * radius;

// Format waktu (mm:ss)
const formattedTime = computed(() => {
  const safeCountdown = currentCountdown.value || 0;
  const minutes = Math.floor(safeCountdown / 60);
  const seconds = safeCountdown % 60;
  return `${minutes.toString().padStart(2, '0')}:${seconds
    .toString()
    .padStart(2, '0')}`;
});

// Hitung dashoffset dinamis (gerak lingkaran)
const dashOffset = computed(() => {
  const progress = currentCountdown.value / initialCountdown.value;
  return circumference * (1 - progress);
});

// Jalankan countdown
onMounted(() => {
  currentCountdown.value = props.countdown || 60;
  initialCountdown.value = props.countdown || 60;
  startCountdown();
});

onUnmounted(() => clearInterval(countdownInterval));

function startCountdown() {
  clearInterval(countdownInterval);
  countdownInterval = setInterval(() => {
    if (currentCountdown.value > 0) {
      currentCountdown.value--;
      timerProgress.value =
        (currentCountdown.value / initialCountdown.value) * 100;

      // mainkan suara tick di 5 detik terakhir
      if (currentCountdown.value <= 5 && currentCountdown.value > 0) {
        if (!audioTick.value) {
          audioTick.value = new Audio('/src/assets/sound_tick.mp3');
        }
        audioTick.value.currentTime = 0;
        audioTick.value.play().catch(() => {});
      }
    } else {
      clearInterval(countdownInterval);
    }
  }, 1000);
}
</script>

<template>
  <div
    class="fixed inset-0 flex flex-col items-center justify-center 
           bg-gradient-to-br from-[#0c2461] to-[#1e3799] 
           text-white z-50 select-none"
  >
    <h2
      class="text-5xl sm:text-6xl font-bold mb-6 text-center 
             drop-shadow-lg tracking-wide"
    >
      Hitung Mundur Iqamah {{ prayerName }}
    </h2>

    <div class="relative w-[360px] h-[360px] sm:w-[440px] sm:h-[440px]">
      <svg
        class="w-full h-full transform -rotate-90"
        viewBox="0 0 400 400"
      >
        <!-- Latar belakang lingkaran -->
        <circle
          cx="200"
          cy="200"
          :r="radius"
          stroke="rgba(255,255,255,0.2)"
          stroke-width="20"
          fill="none"
        />
        <!-- Lingkaran progress -->
        <circle
          cx="200"
          cy="200"
          :r="radius"
          fill="none"
          stroke-linecap="round"
          stroke-width="24"
          stroke="url(#iqamahGradient)"
          :stroke-dasharray="circumference"
          :stroke-dashoffset="dashOffset"
          class="transition-all duration-1000 ease-linear 
                 drop-shadow-[0_0_20px_rgba(255,200,50,0.7)]"
        />
        <defs>
          <linearGradient id="iqamahGradient" x1="0" y1="0" x2="1" y2="1">
            <stop offset="0%" stop-color="#fcbf49" />
            <stop offset="100%" stop-color="#f77f00" />
          </linearGradient>
        </defs>
      </svg>

      <!-- Angka countdown -->
      <div
        class="absolute inset-0 flex items-center justify-center 
               text-[8rem] sm:text-[10rem] font-black tracking-tighter 
               tabular-nums leading-none drop-shadow-lg"
      >
        {{ formattedTime }}
      </div>
    </div>

    <p class="text-2xl sm:text-3xl text-white/80 mt-10 italic">
      "Luruskan dan rapatkan shaf."
    </p>
  </div>
</template>

<style scoped>
svg {
  width: 100%;
  height: 100%;
}
circle {
  transition: stroke-dashoffset 1s linear;
}
</style>
