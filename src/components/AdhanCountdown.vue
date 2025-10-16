<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue';

const props = defineProps({
  countdown: Number,
  prayerName: String,
});

const emit = defineEmits(['countdown-end']);

const phase = ref('countdown'); // 'countdown' atau 'display'
const currentCountdown = ref(props.countdown || 60);
const progressElapsed = ref(0);
const progressDuration = 4 * 60; // 4 menit tampilan adzan
const audioTick = ref(null);

let countdownInterval = null;
let displayInterval = null;

// === PROGRESS RING ===
const radius = 180;
const circumference = 2 * Math.PI * radius;

const dashOffset = computed(() => {
  const progress = currentCountdown.value / props.countdown;
  return circumference * (1 - progress);
});

onMounted(() => {
  startCountdown();
});

onUnmounted(() => {
  clearInterval(countdownInterval);
  clearInterval(displayInterval);
});

function startCountdown() {
  countdownInterval = setInterval(() => {
    currentCountdown.value--;

    // Suara tick 4 detik terakhir
    if (currentCountdown.value <= 4 && currentCountdown.value > 0) {
      if (!audioTick.value) {
        audioTick.value = new Audio('/src/assets/sound_tick.mp3');
      }
      audioTick.value.currentTime = 0;
      audioTick.value.play().catch(e => console.error('Gagal memutar suara:', e));
    }

    // Ketika waktu habis
    if (currentCountdown.value <= 0) {
      clearInterval(countdownInterval);
      phase.value = 'display';

      // Jalankan tampilan adzan berlangsung
      displayInterval = setInterval(() => {
        progressElapsed.value++;
        if (progressElapsed.value >= progressDuration) {
          clearInterval(displayInterval);
          emit('countdown-end');
        }
      }, 1000);
    }
  }, 1000);
}
</script>

<template>
  <!-- === PHASE COUNTDOWN (LINGKARAN EMAS) === -->
  <div
    v-if="phase === 'countdown'"
    class="fixed inset-0 bg-gradient-to-br from-white via-[#f8f9fa] to-[#e9ecef] flex flex-col items-center justify-center z-50 text-slate-900 select-none"
  >
    <h2 class="text-3xl sm:text-5xl md:text-6xl font-bold mb-10 text-center drop-shadow-md px-4">
      Bersiap Menuju Adzan {{ prayerName }}
    </h2>

    <div class="relative flex items-center justify-center w-[65vw] max-w-[480px] aspect-square">
      <!-- SVG LINGKARAN -->
      <svg
        class="w-full h-full transform -rotate-90"
        viewBox="0 0 400 400"
        preserveAspectRatio="xMidYMid meet"
      >
        <circle
          cx="200"
          cy="200"
          :r="radius"
          stroke="rgba(0,0,0,0.08)"
          stroke-width="22"
          fill="none"
        />
        <circle
          cx="200"
          cy="200"
          :r="radius"
          stroke="url(#goldGradient)"
          stroke-width="26"
          fill="none"
          stroke-linecap="round"
          :stroke-dasharray="circumference"
          :stroke-dashoffset="dashOffset"
          class="transition-all duration-1000 ease-linear drop-shadow-[0_0_20px_rgba(255,193,7,0.6)]"
        />

        <defs>
          <linearGradient id="goldGradient" x1="0" y1="0" x2="1" y2="1">
            <stop offset="0%" stop-color="#FFD700" />
            <stop offset="100%" stop-color="#B8860B" />
          </linearGradient>
        </defs>
      </svg>

      <!-- ANGKA COUNTDOWN -->
      <div
        class="absolute text-[20vw] sm:text-[12rem] md:text-[14rem] font-extrabold leading-none tracking-tighter tabular-nums drop-shadow-lg text-slate-900"
      >
        {{ currentCountdown }}
      </div>
    </div>
  </div>

  <!-- === PHASE DISPLAY (ADZAN BERLANGSUNG) === -->
  <div
    v-else-if="phase === 'display'"
    class="fixed inset-0 bg-gradient-to-br from-white via-[#f8f9fa] to-[#e9ecef] flex flex-col items-center justify-center z-50 p-8 text-slate-900"
  >
    <h2 class="text-4xl sm:text-5xl md:text-6xl font-bold mb-8 text-center">
      Sedang Mengumandangkan Adzan {{ prayerName }}
    </h2>
    <p class="text-2xl text-slate-600 mb-16 text-center px-4">
      Hargai dan hormatilah panggilan Allah.
    </p>

    <div class="w-[90%] max-w-4xl bg-gray-200 rounded-full h-5 overflow-hidden shadow-inner">
      <div
        class="bg-gradient-to-r from-amber-400 to-amber-600 h-5 rounded-full transition-all duration-1000 ease-linear"
        :style="{ width: (progressElapsed / progressDuration) * 100 + '%' }"
      ></div>
    </div>
  </div>
</template>
