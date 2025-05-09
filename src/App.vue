<template>
<div ref="container">
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
    <button
      class="bar bar--label bar--yellow"
      @click="toggleFullscreen"
    >
      toggle fullscreen
    </button>
  </div>
  <div
    v-if="timer.duration"
  >
    <div
      style="display: flex; flex-direction: column; height: 100vh; height: calc(100dvh - 16px); margin-bottom: 8px;"
    >
      <div
        v-for="i of bars" :key="i"
        class="bar bar--grow" 
        :class="{
          'bar--green': timer.step >= i*8 && timer.step < 8*BAR_COUNT + i,
          'bar--yellow': timer.step >= 8*BAR_COUNT + i && timer.step < 9*BAR_COUNT + i,
          'bar--red': timer.step >= 9*BAR_COUNT + i,
          'bar--flashing': timer.step >= 10*BAR_COUNT,
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
      @click="startTimer()"
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
  <div class="footer">
    clock v{{ VERSION }} <span v-if="!!BUILD">b{{  BUILD }}</span>
  </div>
</div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { useWakeLock } from '@vueuse/core'
import { useFullscreen } from '@vueuse/core'

  const VERSION = '1.1.0'
  const BUILD = 0

  const el = ref(null)
  const { 
    toggle: toggleFullscreen,
  } = useFullscreen(el)
  const wakeLock = useWakeLock()

  const PRESET_DURATIONS = [2, 5, 10, 15, 25, 30, 45, 60, 90, 120];
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

  const timer = ref<Timer>({...emptyTimer})

  const startTimer = async (duration?: number) => {
    window.scrollTo(0,0);
    if (duration) {
      timer.value.duration = duration;
      timer.value.step = 0;
    }
    timer.value.intervalId = setInterval(
      () => timer.value.step += 1,
      timer.value.duration * 60000 / (10 * BAR_COUNT),
    )
    await lockScreen()
  }

  const onPause = async () => {
    clearInterval(timer.value.intervalId)
    timer.value.intervalId = null;
    await unlockScreen();
  }

  const onBack = async () => {
    reset();
    await unlockScreen();
  }

  const lockScreen = async () => {
    if (wakeLock.isSupported.value && !wakeLock.isActive.value) {
      try {
        await wakeLock.request('screen')
        
      } catch (error) {
        console.error('Failed to acquire wake lock:', error)
      }
    }
  }

  const unlockScreen = async () => {
    if (wakeLock.isSupported.value && wakeLock.isActive.value) {
      await wakeLock.release();
    }
  }

  const reset = () => {
    clearInterval(timer.value.intervalId)
    timer.value = {...emptyTimer}
  }

</script>
