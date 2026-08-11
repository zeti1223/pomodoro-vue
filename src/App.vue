<script setup>
import { ref, computed, onUnmounted } from "vue";

const WORK_TIME = 25 * 60; // 25 minutes in seconds
const timeLeft = ref(WORK_TIME);
const isRunning = ref(false);
let timerInterval = null;

const formattedTime = computed(() => {
  const minutes = Math.floor(timeLeft.value / 60);
  const seconds = timeLeft.value % 60;
  return `${minutes.toString().padStart(2, "0")}:${seconds.toString().padStart(2, "0")}`;
});

const startTimer = () => {
  if (isRunning.value) return;
  isRunning.value = true;
  timerInterval = setInterval(() => {
    if (timeLeft.value > 0) {
      timeLeft.value--;
    } else {
      clearInterval(timerInterval);
      isRunning.value = false;
    }
  }, 1000);
};

const pauseTimer = () => {
  isRunning.value = false;
  clearInterval(timerInterval);
};

const resetTimer = () => {
  pauseTimer();
  timeLeft.value = WORK_TIME;
};

onUnmounted(() => {
  clearInterval(timerInterval);
});
</script>

<template>
  <div class="pomodoro-container">
    <h1>Pomodoro Timer</h1>
    <div class="timer-display">
      {{ formattedTime }}
    </div>
    <div class="controls">
      <button @click="startTimer" :disabled="isRunning">Start</button>
      <button @click="pauseTimer" :disabled="!isRunning">Pause</button>
      <button @click="resetTimer">Reset</button>
    </div>
  </div>
</template>

<style scoped>
.pomodoro-container {
  text-align: center;
  font-family: sans-serif;
  margin-top: 50px;
}
.timer-display {
  font-size: 4rem;
  font-weight: bold;
  margin: 20px 0;
}
button {
  margin: 0 10px;
  padding: 10px 20px;
  font-size: 1.2rem;
  cursor: pointer;
}
</style>
