<script setup>
import { ref, onMounted, computed } from 'vue';
import TheHeader from './components/TheHeader.vue';
import PrayerTimes from './components/PrayerTimes.vue';
import MainDisplay from './components/MainDisplay.vue';
import DynamicContent from './components/DynamicContent.vue';
import TheFooter from './components/TheFooter.vue';
import AdhanCountdown from './components/AdhanCountdown.vue';
import IqomahCountdown from './components/IqomahCountdown.vue';
import SholatInProgress from './components/SholatInProgress.vue';

// --- KONFIGURASI ---
const SPREADSHEET_ID = '1nyw_SmnwTd6Er1rBhosZe11eAyKooKFRRm-KuYsPLdQ';
const hijriMonthsID = { 1: 'Muharram', 2: 'Safar', 3: 'Rabiul Awal', 4: 'Rabiul Akhir', 5: 'Jumadil Awal', 6: 'Jumadil Akhir', 7: 'Rajab', 8: "Sya'ban", 9: 'Ramadhan', 10: 'Syawal', 11: "Dzulqa'dah", 12: 'Dzulhijjah' };
const iqomahDurations = { Subuh: 7, Dzuhur: 10, Ashar: 10, Maghrib: 7, Isya: 7 };
const sholatDuration = 4 * 60;

// --- STATE ---
const appState = ref('loading');
const currentTime = ref(new Date());
const prayerData = ref(null);
const hijriDate = ref(null);
const currentPrayerEvent = ref({ name: null, timer: 0, duration: 0, elapsed: 0 });

// --- STATE KONTEN ---
const asmaulHusnaList = ref([]);
const financialReport = ref([]);
const runningText = ref('');
const fullRotationList = ref([]);
const currentContentIndex = ref(0);
const cloudinaryImages = ref([]);

const nextPrayerInfo = computed(() => {
  if (!prayerData.value) return null;

  const now = currentTime.value;
  const schedule = prayerData.value;

  // helper: ambil HH:MM dari string waktu (toleran terhadap tambahan teks)
  function parseHM(t) {
    if (!t) return null;
    const match = String(t).match(/(\d{1,2}):(\d{2})/);
    if (!match) return null;
    const hh = parseInt(match[1], 10);
    const mm = parseInt(match[2], 10);
    if (isNaN(hh) || isNaN(mm)) return null;
    return { hh, mm };
  }

  const prayerTimesToday = [
    { name: 'Subuh', key: 'subuh', time: schedule.subuh },
    { name: 'Dzuhur', key: 'dzuhur', time: schedule.dzuhur },
    { name: 'Ashar', key: 'ashar', time: schedule.ashar },
    { name: 'Maghrib', key: 'maghrib', time: schedule.maghrib },
    { name: 'Isya', key: 'isya', time: schedule.isya },
  ];

  // cari jadwal hari ini yang masih di depan sekarang
  for (const prayer of prayerTimesToday) {
    const parsed = parseHM(prayer.time);
    if (!parsed) continue;
    const prayerTime = new Date(now.getFullYear(), now.getMonth(), now.getDate(), parsed.hh, parsed.mm, 0, 0);
    if (prayerTime.getTime() > now.getTime()) {
      return { name: prayer.name, countdown: Math.floor((prayerTime - now) / 1000) };
    }
  }

  // Jika tidak menemukan (artinya sudah lewat Isya hari ini), fallback ke Subuh besok
  const subuhParsed = parseHM(schedule.subuh);
  if (subuhParsed) {
    const tomorrow = new Date(now);
    tomorrow.setDate(now.getDate() + 1);
    const subuhTomorrow = new Date(tomorrow.getFullYear(), tomorrow.getMonth(), tomorrow.getDate(), subuhParsed.hh, subuhParsed.mm, 0, 0);
    return { name: 'Subuh', countdown: Math.floor((subuhTomorrow - now) / 1000) };
  }

  // kalau belum ada data subuh sama sekali
  return null;
});

const activeContent = computed(() => fullRotationList.value[currentContentIndex.value]);

function startIqomahCountdown() {
  const prayerName = currentPrayerEvent.value.name;
  const duration = (iqomahDurations[prayerName] || 7) * 60;
  currentPrayerEvent.value = { name: prayerName, duration, timer: duration, elapsed: 0 };
  appState.value = 'iqomahCountdown';
}
function startSholatInProgress() {
  currentPrayerEvent.value = { ...currentPrayerEvent.value, duration: sholatDuration, timer: sholatDuration, elapsed: 0 };
  appState.value = 'sholatInProgress';
}

// --- FUNGSI FETCH SPREADSHEET ---
async function fetchGoogleSheet(sheetName) {
  try {
    const res = await fetch(`https://docs.google.com/spreadsheets/d/${SPREADSHEET_ID}/gviz/tq?tqx=out:json&sheet=${sheetName}&cacheBust=${Date.now()}`);
    const text = await res.text();
    const json = JSON.parse(text.substring(47).slice(0, -2));
    return json.table.rows.map(row => row.c);
  } catch (err) {
    console.error(`Error fetching sheet ${sheetName}:`, err);
    return [];
  }
}

async function fetchSheetData() {
  try {
    const [financeRows, infoRows, galleryRows] = await Promise.all([
      fetchGoogleSheet('Laporan_Keuangan'),
      fetchGoogleSheet('Info_Utama'),
      fetchGoogleSheet('Galeri_Gambar')
    ]);

    // --- GAMBAR DARI SPREADSHEET ---
    const newImages = galleryRows
      .map(row => ({
        Judul: (row[0]?.v || '').trim(),
        URL_Cloudinary: (row[1]?.v || '').trim(),
        Kategori: (row[2]?.v || '').trim()
      }))
      .filter(img => img.URL_Cloudinary && img.URL_Cloudinary.startsWith('http'));

    if (newImages.length) {
      cloudinaryImages.value = newImages;
      localStorage.setItem('cloudinaryImages', JSON.stringify(newImages));
    }

    // --- LAPORAN KEUANGAN ---
    financialReport.value = financeRows
      .filter(r => r && r[2]?.v?.toLowerCase() === 'ya')
      .map(r => {
        const amount = parseFloat(String(r[1]?.v || '0').replace(/[^0-9]/g, '')) || 0;
        return {
          keterangan: r[0]?.v,
          jumlah: new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(amount)
        };
      });

    // --- RUNNING TEXT ---
    const rtRow = infoRows.find(r => r[0]?.v === 'Running Text');
    if (rtRow && rtRow[1]?.v) runningText.value = rtRow[1].v;

  } catch (error) {
    console.error('❌ Gagal memuat data spreadsheet:', error);
  }
}

// --- FETCH DATA UTAMA ---
async function fetchInitialData() {
  try {
    const now = new Date();
    const y = now.getFullYear(), m = now.getMonth() + 1, d = now.getDate();
    const lat = -6.8715, lng = 107.5767;

    const [prayerRes, dateRes, asmaulRes] = await Promise.all([
      fetch(`https://api.aladhan.com/v1/timings/${y}-${m}-${d}?latitude=${lat}&longitude=${lng}&method=20&timezonestring=Asia/Jakarta`),
      fetch(`https://api.aladhan.com/v1/gToH?date=${d}-${m}-${y}`),
      fetch('https://api.myquran.com/v2/husna/semua')
    ]);

    const prayerJson = await prayerRes.json();
    const dateJson = await dateRes.json();
    const asmaulData = await asmaulRes.json();

    // Jadwal sholat
    const t = prayerJson.data.timings;
    prayerData.value = {
      imsak: t.Imsak, subuh: t.Fajr, terbit: t.Sunrise, dzuhur: t.Dhuhr,
      ashar: t.Asr, maghrib: t.Maghrib, isya: t.Isha
    };

    // Hijriyah
    const h = dateJson.data.hijri;
    hijriDate.value = {
      masehi: { tanggal: now.toLocaleDateString('id-ID', { weekday: 'long', day: 'numeric', month: 'long', year: 'numeric' }) },
      hijri: { tanggal: `${h.day} ${hijriMonthsID[h.month.number]} ${h.year} H` }
    };

    // Asmaul Husna
    asmaulHusnaList.value = asmaulData.data.map(a => ({
      type: 'asmaulHusna',
      data: { arab: a.arab, latin: a.latin, indo: a.indo }
    }));

    appState.value = 'normal';
  } catch (err) {
    console.error('❌ Gagal mengambil data awal:', err);
    appState.value = 'error';
  }
}

// --- REBUILD LIST HANYA ASMAUL HUSNA ---
function rebuildFullRotationList() {
  const asmaul = asmaulHusnaList.value || [];
  fullRotationList.value = [...asmaul];
}

// --- ONMOUNTED ---
onMounted(async () => {
  console.log('🚀 App mounted');
  
  // 🖥️ AUTO FULLSCREEN DENGAN INTERAKSI USER
  function enableFullscreen() {
    const el = document.documentElement;
    if (el.requestFullscreen) {
      el.requestFullscreen().then(() => {
        console.log('✅ Fullscreen aktif');
        window.removeEventListener('click', enableFullscreen);
      }).catch(err => {
        console.warn('⚠️ Gagal masuk fullscreen:', err);
      });
    }
  }

  if (document.fullscreenEnabled && !document.fullscreenElement) {
    window.addEventListener('click', enableFullscreen, { once: true });
    console.log('🖱️ Klik layar untuk mengaktifkan fullscreen.');
  }

  const cached = localStorage.getItem('cloudinaryImages');
  if (cached) cloudinaryImages.value = JSON.parse(cached);

  await Promise.all([fetchInitialData(), fetchSheetData()]);
  rebuildFullRotationList();

  // Rotasi konten Asmaul Husna
  setInterval(() => {
    if (fullRotationList.value.length)
      currentContentIndex.value = (currentContentIndex.value + 1) % fullRotationList.value.length;
  }, 15000);

  // Refresh data tiap 5 menit
  setInterval(async () => {
    console.log('🔄 Auto refresh data spreadsheet');
    await fetchSheetData();
    cloudinaryImages.value = [...cloudinaryImages.value];
  }, 5 * 60 * 1000);

  // Keyboard “R” = Refresh manual
  window.addEventListener('keydown', async (e) => {
    if (e.key.toLowerCase() === 'r') {
      console.log('🔁 Manual refresh by R');
      await fetchSheetData();
    }
  });

  // Timer jam & logika adzan + auto refresh harian
  setInterval(async () => {
    const now = new Date();
    currentTime.value = now;

    if (now.getHours() === 0 && now.getMinutes() === 0 && now.getSeconds() < 5) {
      console.log('🌅 Hari baru terdeteksi, memuat ulang data...');
      await fetchInitialData();
      await fetchSheetData();
      rebuildFullRotationList();
    }

    if (appState.value === 'normal') {
      if (nextPrayerInfo.value && nextPrayerInfo.value.countdown <= 60) {
        currentPrayerEvent.value.name = nextPrayerInfo.value.name;
        appState.value = 'adhanCountdown';
      }
    } else if (['iqomahCountdown', 'sholatInProgress'].includes(appState.value)) {
      currentPrayerEvent.value.timer--;
      currentPrayerEvent.value.elapsed++;
      if (currentPrayerEvent.value.timer <= 0) {
        if (appState.value === 'iqomahCountdown') startSholatInProgress();
        else appState.value = 'normal';
      }
    }
  }, 1000);
});
</script>



<template>
  <div> <div v-if="appState === 'normal'" class="h-screen w-screen flex flex-col font-sans bg-gradient-to-br from-gray-50 to-white text-slate-800 overflow-hidden">
      <TheHeader :masehiDate="hijriDate?.masehi?.tanggal" :hijriDateStr="hijriDate?.hijri?.tanggal" />
      <main class="flex flex-1 overflow-hidden items-stretch">
        <div class="flex-shrink-0 w-[260px] xl:w-[300px] bg-white rounded-2xl border border-gray-200 shadow-lg overflow-hidden flex flex-col">
          <h2 class="text-center py-2 font-bold text-lg border-b border-gray-200">Jadwal Sholat</h2>
          <div class="flex-1 overflow-y-auto p-3">
            <PrayerTimes :schedule="prayerData" :nextPrayerName="nextPrayerInfo?.name" />
          </div>
        </div>
        <div class="flex-1 flex flex-col gap-4 overflow-hidden">
          <div v-if="nextPrayerInfo" class="h-24 bg-white rounded-xl flex items-center justify-between px-8 shadow-lg border border-gray-200">
            <div class="text-center w-48">
              <div class="text-lg text-[#B8860B] font-medium">Menuju Waktu</div> <div class="text-3xl font-bold tracking-wider">{{ nextPrayerInfo.name }}</div>
            </div>
            <div class="text-6xl sm:text-7xl font-black text-slate-800 tracking-widest tabular-nums drop-shadow-sm">
              {{ currentTime.toLocaleTimeString('en-GB') }}
            </div>
            <div class="text-center w-48">
              <div class="text-lg text-[#B8860B] font-medium">Dalam</div> <div class="text-3xl font-bold tracking-wider tabular-nums drop-shadow-sm">
                {{ new Date(nextPrayerInfo.countdown * 1000).toISOString().substr(11, 8) }}
              </div>
            </div>
          </div>
          <div class="flex-1 overflow-hidden rounded-xl border border-gray-200 shadow-inner">
            <MainDisplay :images="cloudinaryImages" />
          </div>
        </div>
        <div class="flex-shrink-0 w-[300px] bg-white rounded-2xl border border-gray-200 shadow-lg overflow-hidden">
          <DynamicContent :activeContent="activeContent" :financialReport="financialReport" />
        </div>
      </main>
      <TheFooter :text="runningText" />
    </div>

    <AdhanCountdown
      v-if="appState === 'adhanCountdown'"
      :countdown="nextPrayerInfo?.countdown ?? 60"
      :prayerName="currentPrayerEvent.name"
      @countdown-end="startIqomahCountdown"
    />

    <IqomahCountdown
      v-if="appState === 'iqomahCountdown'"
      :countdown="currentPrayerEvent.timer"
      :prayerName="currentPrayerEvent.name"
    />

    <SholatInProgress
      v-if="appState === 'sholatInProgress'"
      :prayerName="currentPrayerEvent.name"
      :duration="currentPrayerEvent.duration"
      :elapsed="currentPrayerEvent.elapsed"
    />
  </div>
</template>



<style>
@import url('https://fonts.googleapis.com/css2?family=Amiri&family=Inter:wght@400;500;700;900&display=swap');
.font-sans { font-family: 'Inter', sans-serif; }
.font-arabic { font-family: 'Amiri', serif; }
.tabular-nums { font-feature-settings: 'tnum' on, 'lnum' on; }

/* Efek Glassmorphism */
.bg-black\/20 {
  background: rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
}

/* Animasi loading */
.animate-pulse {
  animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}
</style>