<script setup>
import { computed, ref } from "vue";
const milliseconde = ref(0);
const isRunning = ref(false);
const intervalId = ref(null);

function startTimer() {
  if (isRunning.value) {
    return;
  }
  isRunning.value = true;
  intervalId.value = setInterval(() => {
    milliseconde.value += 10;
  }, 10);
}
function pausemilliseconder() {
  clearInterval(intervalId.value);
  isRunning.value = false
}
const ms = computed(() => {
  return Math.floor((milliseconde.value % 1000) / 10);
});
const seconde = computed(()=>{
    return Math.floor( (milliseconde.value/1000) % 60 )
})
const minutes = computed(()=>{
    return Math.floor( (milliseconde.value / 60000) % 60 )
})
const hours =  computed(()=>{
    return Math.floor( (milliseconde.value / 3600000) % 60 )
})
</script>
<template>
  <div
    class="bg-[#AA00FF] w-100 h-70 flex-col items-center justify-between rounded-md flex p-5"
  >
    <!-- EN TÊTE -->
    <div>
      <span class="text-5xl">{{ String(hours).padStart(2,'0')}} : {{ String(minutes).padStart(2,'0') }} 
        : {{ String(seconde).padStart(2,'0') }} : {{ String(ms).padStart(2,'0') }}</span>
    </div>
    <!-- CONTROL -->
    <div>
      <div class="flex flex-row gap-4">
        <button
          class="p-2 rounded-md bg-blue-100 cursor-pointer"
          @click="startTimer"
        >
          <span class="tracking-[3px]">START</span>
        </button>
        <button
          class="p-2 rounded-md bg-blue-100 cursor-pointer"
          @click="pausemilliseconder"
        >
          <span class="tracking-[3px]">PAUSE</span>
        </button>
        <button class="p-2 rounded-md bg-blue-100 cursor-pointer">
          <span class="tracking-[3px]">STOP</span>
        </button>
      </div>
    </div>
  </div>
</template>
