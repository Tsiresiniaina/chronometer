<script setup>
import { computed, onBeforeUnmount, ref } from "vue";
import { Play, Pause, Flag, RotateCcw, } from "@lucide/vue";
import { build } from "vite";
const time = ref(0);
const isRunning = ref(false);
const intervalId = ref(null);

// GESTION DES TOURS
const laps = ref([]);

let previousLapTime = 0;
let startedAt = 0;

function addLap() {
  if(!isRunning.value){
    return
  }
  const lapTime = time.value - previousLapTime;
  const objectLap = {
    lapGrade: laps.value.length + 1,
    lapTime: lapTime,
    totalTime: time.value,
  };
  laps.value.push(objectLap);
  previousLapTime = time.value;
}
function startTimer() {
  if (isRunning.value) {
    return;
  }
  isRunning.value = true;
  //Permet de reprendre là où était réellement après une pause
  startedAt = performance.now() - time.value
  intervalId.value = setInterval(() => {
    time.value = performance.now() - startedAt;
  }, 10);
}
function pauseTimer() {
  if(!isRunning.value){
    return
  }
  clearInterval(intervalId.value);
  intervalId.value=null
  isRunning.value = false;
}
function resetTimer() {
  time.value = 0;
  laps.value = [];
  pauseTimer();
  previousLapTime = 0;
}
function formatLapTime(lapTime) {
  let stringTime = "";
  const seconde = Math.floor((lapTime / 1000) % 60);
  const minutes = Math.floor((lapTime / 60000) % 60);
  const hours = Math.floor((lapTime / 3600000) % 60);
  const ms = Math.floor((lapTime % 1000) / 10);
  stringTime = `${String(hours).padStart(2, "0")}:${String(minutes).padStart(2, "0")}:${String(seconde).padStart(2, "0")}.${String(ms).padStart(2, "0")}`;
  return stringTime;
}
const ms = computed(() => {
  return Math.floor((time.value % 1000) / 10);
});
const seconde = computed(() => {
  return Math.floor((time.value / 1000) % 60);
});
const minutes = computed(() => {
  return Math.floor((time.value / 60000) % 60);
});
const hours = computed(() => {
  return Math.floor((time.value / 3600000) % 60);
});
onBeforeUnmount(()=>{
  clearInterval(intervalId.value)
})
</script>
<template>
  <div
    class="w-full max-w-xl mx-auto bg-[#F4ECD7] rounded-2xl p-5 sm:p-6 shadow-2xl border border-white/10 flex flex-col gap-6"
  >
    <!-- AFFICHAGE DU TEMPS -->
    <div class="bg-[#FCF5EB] rounded-2xl p-5 sm:p-7 border border-white/5">
      <p class="text-center text-gray-500 text-xs tracking-[0.2em] mb-4">
        TEMPS ÉCOULÉ
      </p>

      <div
        class="flex items-center justify-center font-mono font-bold text-3xl sm:text-5xl text-[#1D1814] tracking-tight tabular-nums whitespace-nowrap"
      >
        <span>{{ String(hours).padStart(2, "0") }}</span>
        <span class="text-gray-300 mx-1">:</span>
        <span>{{ String(minutes).padStart(2, "0") }}</span>
        <span class="text-gray-300 mx-1">:</span>
        <span>{{ String(seconde).padStart(2, "0") }}</span>
        <span class="text-[#AF7748] mx-1">.</span>
        <span class="text-[#AF7748]">{{ String(ms).padStart(2, "0") }}</span>
      </div>
      <div class="flex justify-center mt-5">
        <span
          class="text-xs text-[#AF855B] bg-[#F0E6DD] px-3 py-1 rounded-full"
        >
          {{ isRunning ? "Chronomètre en cours" : "En attente" }}
        </span>
      </div>
    </div>

    <!-- COMMANDES -->
    <div class="grid grid-cols-4 gap-3">
      <button
        @click="startTimer"
        :disabled="isRunning"
        class="flex flex-col items-center gap-2 p-3 rounded-xl cursor-pointer bg-[#af7748] text-[#FCF5EB] hover:bg-[#d18954] disabled:opacity-40 disabled:cursor-not-allowed transition-all"
      >
        <Play :size="22" />
        <span class="text-xs font-medium">Start</span>
      </button>

      <button
        @click="pauseTimer"
        :disabled="!isRunning"
        class="flex flex-col items-center gap-2 p-3 rounded-xl cursor-pointer bg-[#af7748] text-[#FCF5EB] hover:bg-[#d18954] disabled:opacity-40 disabled:cursor-not-allowed transition-all"
      >
        <Pause :size="22" />
        <span class="text-xs font-medium">Pause</span>
      </button>

      <button
        @click="addLap"
        :disabled="!isRunning"
        class="flex flex-col items-center gap-2 p-3 rounded-xl cursor-pointer bg-[#af7748] text-[#FCF5EB] hover:bg-[#d18954] disabled:opacity-40 disabled:cursor-not-allowed transition-all"
      >
        <Flag :size="22" />
        <span class="text-xs font-medium">Lap</span>
      </button>

      <button
        @click="resetTimer"
        class="flex flex-col items-center gap-2 p-3 rounded-xl cursor-pointer bg-[#af7748] text-[#FCF5EB] hover:bg-[#d18954] disabled:opacity-40 disabled:cursor-not-allowed transition-all"
      >
        <RotateCcw :size="22" />
        <span class="text-xs font-medium">Reset</span>
      </button>
    </div>

    <!-- LISTE DES TOURS -->
    <div
      v-if="laps.length"
      class="bg-[#FCF5EB] rounded-2xl border border-white/5 overflow-hidden"
    >
      <!-- HEADER DE LA LISTE -->
      <div
        class="flex items-center justify-between p-4 border-b border-white/5"
      >
        <div class="flex items-center gap-3">
          <div>
            <h3 class="text-[#AF7748] font-semibold text-sm">
              Historique des tours
            </h3>
            <p v-if="laps.length == 1" class="text-gray-400 text-xs">
              {{ laps.length }} tour enregistré
            </p>
            <p v-else class="text-gray-500 text-xs">
              {{ laps.length }} tours enregistrés
            </p>
          </div>
        </div>
      </div>

      <!-- TABLEAU -->
      <div class="max-h-65 overflow-y-auto scroll-smooth scrollbar-thin scrollbar-thumb-[#D8BD9E]">
        <table class="w-full text-sm text-left">
          <thead class="sticky top-0 z-20 bg-[#FCF5EB] text-gray-400 text-[11px] uppercase tracking-wider">
            <tr>
              <th class="px-4 py-3 font-medium">
                {{ laps.length == 1 ? "TOUR" : "TOURS" }}
              </th>
              <th class="px-4 py-3 font-medium text-center">Temps du tour</th>
              <th class="px-4 py-3 font-medium text-right">Temps total</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="lap in [...laps].reverse()"
              :key="lap.lapGrade"
              class="group/ligne border-t border-white/5 hover:bg-[#AF7748] transition-colors cursor-pointer"
            >
              <td class="px-4 py-3">
                <span
                  class="w-7 h-7 inline-flex items-center justify-center 
                  rounded-lg text-[#AF7748] group-hover/ligne:text-[#FCF5EB] text-xs font-bold"
                >
                  {{ lap.lapGrade }}
                </span>
              </td>

              <td
                class="px-4 py-3 text-center font-mono font-semibold text-[#AF7748]
                 group-hover/ligne:text-[#FCF5EB] tabular-nums"
              >
                {{ formatLapTime(lap.lapTime) }}
              </td>
              <td
                class="px-4 py-3 text-right font-mono text-[#AF7748] 
                group-hover/ligne:text-[#FCF5EB] tabular-nums"
              >
                {{ formatLapTime(lap.totalTime) }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <!-- ÉTAT VIDE -->
    <div
      v-else
      class="flex flex-col items-center justify-center py-5 text-gray-400"
    >
      <p class="text-sm">Aucun tour enregistré</p>
      <p class="text-xs mt-1">Appuie sur Lap pour enregistrer un tour</p>
    </div>
  </div>
</template>
