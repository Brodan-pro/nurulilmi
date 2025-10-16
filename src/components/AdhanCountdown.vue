<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue';

const props = defineProps({
  countdown: Number,
  prayerName: String,
});

const emit = defineEmits(['countdown-end']);

const phase = ref('countdown'); // 'countdown' atau 'display'
const currentCountdown = ref(props.countdown);
const progressElapsed = ref(0);
const progressDuration = 4 * 60; // 4 menit
let countdownInterval = null;
let displayInterval = null;
const audioTick = ref(null);

// === PROGRESS RING ===
const radius = 180; // ukuran radius lingkaran
const circumference = 2 * Math.PI * radius;
const dashOffset = computed(() => {
  const progress = currentCountdown.value / props.countdown;
  return circumference * (1 - progress);
});

onMounted(() => {
  countdownInterval = setInterval(() => {
    currentCountdown.value--;

    // Suara tick 3 detik terakhir
    if (currentCountdown.value <= 4 && currentCountdown.value > 0) {
      if (!audioTick.value) {
        audioTick.value = new Audio('/src/assets/sound_tick.mp3');
      }
      audioTick.value.currentTime = 0;
      audioTick.value.play().catch(e => console.error("Gagal memutar suara:", e));
    }

    // Jika hitungan selesai
    if (currentCountdown.value <= 0) {
      clearInterval(countdownInterval);
      phase.value = 'display';

      // Mulai tampilan “Sedang Mengumandangkan Adzan”
      displayInterval = setInterval(() => {
        progressElapsed.value++;
        if (progressElapsed.value >= progressDuration) {
          clearInterval(displayInterval);
          emit('countdown-end');
        }
      }, 1000);
    }
  }, 1000);
});

onUnmounted(() => {
  if (countdownInterval) clearInterval(countdownInterval);
  if (displayInterval) clearInterval(displayInterval);
});
</script>

<template>
  <!-- === PHASE COUNTDOWN (LINGKARAN) === -->
  <div
    v-if="phase === 'countdown'"
    class="fixed inset-0 bg-gradient-to-br from-[#0c2461] to-[#1e3799] flex flex-col items-center justify-center z-50 text-white select-none"
  >
    <h2 class="text-6xl font-bold mb-6 text-center drop-shadow-md">
      Bersiap Menuju Adzan {{ prayerName }}
    </h2>

    <div class="relative flex items-center justify-center">
      <!-- SVG LINGKARAN -->
      <svg
        class="w-[480px] h-[480px] transform -rotate-90"
        viewBox="0 0 400 400"
      >
        <!-- Background lingkaran -->
        <circle
          cx="200"
          cy="200"
          :r="radius"
          stroke="rgba(255,255,255,0.2)"
          stroke-width="20"
          fill="none"
        />
        <!-- Progress lingkaran -->
        <circle
          cx="200"
          cy="200"
          :r="radius"
          stroke="url(#gradient)"
          stroke-width="24"
          fill="none"
          stroke-linecap="round"
          :stroke-dasharray="circumference"
          :stroke-dashoffset="dashOffset"
          class="transition-all duration-1000 ease-linear drop-shadow-[0_0_15px_rgba(0,150,255,0.6)]"
        />
        <defs>
          <linearGradient id="gradient" x1="0" y1="0" x2="1" y2="1">
            <stop offset="0%" stop-color="#00b4d8" />
            <stop offset="100%" stop-color="#0077b6" />
          </linearGradient>
        </defs>
      </svg>

      <!-- ANGKA COUNTDOWN -->
      <div class="absolute text-[15rem] font-black leading-none tracking-tighter drop-shadow-xl">
        {{ currentCountdown }}
      </div>
    </div>
  </div>

  <!-- === PHASE DISPLAY (ADZAN BERLANGSUNG) === -->
  <div
    v-else-if="phase === 'display'"
    class="fixed inset-0 bg-gradient-to-br from-gray-50 to-white flex flex-col items-center justify-center z-50 p-8"
  >
    <h2 class="text-6xl font-bold text-slate-800 mb-8">
      Sedang Mengumandangkan Adzan {{ prayerName }}
    </h2>
    <p class="text-3xl text-slate-600 mb-16">
      Hargai dan hormatilah panggilan Allah.
    </p>

    <!-- Progress Bar (biarkan seperti semula) -->
    <div class="w-full max-w-4xl bg-gray-200 rounded-full h-4 overflow-hidden">
      <div
        class="bg-gradient-to-r from-amber-400 to-amber-600 h-4 rounded-full transition-all duration-1000 ease-linear"
        :style="{ width: (progressElapsed / progressDuration) * 100 + '%' }"
      ></div>
    </div>
  </div>
</template>
