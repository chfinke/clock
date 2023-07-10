<template>
  <div
    v-if="!timer.duration"
  >
    <button
      v-for="duration in PRESET_DURATIONS" :key=duration
      class="bar bar--label bar--green"
      @click="startTimer(duration)"
    >
      {{ duration }}
    </button>
  </div>
  <div
    v-if="timer.duration"
  >
    <div
      style="display: flex; flex-direction: column; height: 100vh; height: calc(100dvh - 16px); margin-bottom: 8px;"
    >
      <!-- <span>[{{ timer }}]</span> -->
      <div
        v-for="i of bars" :key="i"
        class="bar bar--grow" 
        :class="{
          'bar--green': timer.step >= i*4 && timer.step < 4*BAR_COUNT + i,
          'bar--yellow': timer.step >= 4*BAR_COUNT + i && timer.step < 5*BAR_COUNT + i,
          'bar--red': timer.step >= 5*BAR_COUNT + i,
          'bar--flashing': timer.step >= 6*BAR_COUNT,
        }"
      ></div>
    </div>
    <button
      v-if="timer.intervalId"
      class="bar bar--label bar--yellow" 
      @click="onPause"
    >
      pause
    </button>
    <button
      v-if="!timer.intervalId"
      class="bar bar--label bar--green" 
      @click="onResume"
    >
      resume
    </button>
    <button 
      class="bar bar--label bar--red" 
      @click="onBack"
    >
      back
    </button>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";

  const PRESET_DURATIONS = [5, 10, 15, 25, 30, 45, 60, 90, 120];
  const BAR_COUNT = 15;

  const bars: number[] = [];
  for (let i = BAR_COUNT; i > 0; i--) {
    bars.push(i);
  }


  interface Timer {
    duration: number;
    step: number;
    intervalId: any;
  }

  const emptyTimer: Timer = {
    duration: 0,
    step: 0,
    intervalId: null,
  }

  const timer = ref<Timer>(emptyTimer)

  const startTimer = (duration: number) => {
    window.scrollTo(0,0);
    timer.value = {
        duration,
        step: 0,
        intervalId: setInterval(
          () => timer.value.step += 1,
          duration * 60000 / (6 * BAR_COUNT),
        )
    };
    
  }

  const onPause = () => {
    clearInterval(timer.value.intervalId)
    timer.value.intervalId = null;
  }

  const onResume = () => {
    window.scrollTo(0,0);
    timer.value.intervalId = setInterval(
      () => timer.value.step += 1,
      timer.value.duration * 60000 / (6 * BAR_COUNT),
    )
  }

  const onBack = () => {
    clearInterval(timer.value.intervalId)
    timer.value = emptyTimer;
  }
</script>
