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

const progressPercent = computed(() => {
  return (timeLeft.value / WORK_TIME) * 100;
});

const startTimer = () => {
  if (isRunning.value) return;
  isRunning.value = true;
  timerInterval = setInterval(() => {
    if (timeLeft.value > 0) {
      timeLeft.value--;
    } else {
      pauseTimer();
      alert("Time Is Up!");
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
  <main class="app-wrapper">
    <div class="pomodoro-card">
      <header>
        <h1>Pomodoro</h1>
        <div class="status-dot" :class="{ active: isRunning }"></div>
      </header>

      <div class="timer-circle">
        <svg class="progress-ring" width="240" height="240">
          <circle
            class="progress-ring_bg"
            stroke-width="8"
            fill="transparent"
            r="110"
            cx="120"
            cy="120"
          />
          <circle
            class="progress-ring_circle"
            stroke-width="8"
            fill="transparent"
            r="110"
            cx="120"
            cy="120"
            :style="{ strokeDashoffset: 691 - (691 * progressPercent) / 100 }"
          />
        </svg>
        <div class="time-display">{{ formattedTime }}</div>
      </div>

      <div class="controls">
        <button class="btn btn-icon" @click="resetTimer" title="Restart">
          <svg
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8" />
            <path d="M3 3v5h5" />
          </svg>
        </button>

        <button
          class="btn btn-main"
          @click="isRunning ? pauseTimer() : startTimer()"
        >
          <svg
            v-if="!isRunning"
            viewBox="0 0 24 24"
            fill="currentColor"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <polygon points="5 3 19 12 5 21 5 3" />
          </svg>
          <svg
            v-else
            viewBox="0 0 24 24"
            fill="currentColor"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <rect x="6" y="4" width="4" height="16" />
            <rect x="14" y="4" width="4" height="16" />
          </svg>
        </button>

        <button class="btn btn-icon disabled" title="Settings">
          <svg
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <circle cx="12" cy="12" r="3" />
            <path
              d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06-.06a1.65 1.65 0 0 0 .33-1.82 1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06a1.65 1.65 0 0 0 1.82.33H9a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06a1.65 1.65 0 0 0-.33 1.82V9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z"
            />
          </svg>
        </button>
      </div>
    </div>
  </main>
</template>

<style>
body {
  margin: 0;
  background-color: #11111b;
}
</style>

<style scoped>
.app-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  font-family:
    system-ui,
    -apple-system,
    sans-serif;
  color: #cdd6f4;
}

.pomodoro-card {
  background-color: #1e1e2e;
  padding: 3rem 2.5rem;
  border-radius: 32px;
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.5);
  border: 1px solid #313244;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2.5rem;
  width: 100%;
  max-width: 360px;
}

header {
  display: flex;
  align-items: center;
  gap: 12px;
}

h1 {
  margin: 0;
  font-size: 1.2rem;
  font-weight: 700;
  color: #b4befe;
  letter-spacing: 2px;
  text-transform: uppercase;
}

.status-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background-color: #f38ba8;
  box-shadow: 0 0 8px #f38ba8;
  transition: all 0.3s ease;
}

.status-dot.active {
  background-color: #a6e3a1;
  box-shadow: 0 0 12px #a6e3a1;
}


.timer-circle {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
}

.progress-ring {
  transform: rotate(-90deg);
}

.progress-ring_bg {
  stroke: #313244;
}

.progress-ring_circle {
  stroke: #b4befe;
  stroke-dasharray: 691; /* 2 * PI * R (110) */
  transition: stroke-dashoffset 1s linear;
  stroke-linecap: round;
}

.time-display {
  position: absolute;
  font-size: 4rem;
  font-weight: 700;
  font-variant-numeric: tabular-nums;
  color: #cdd6f4;
}

.controls {
  display: flex;
  align-items: center;
  gap: 1.5rem;
}

.btn {
  background: none;
  border: none;
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  color: #a6adc8;
}

.btn:hover {
  color: #cdd6f4;
}

.btn:active {
  transform: scale(0.92);
}

.btn-icon {
  width: 52px;
  height: 52px;
  border-radius: 50%;
  background-color: #181825;
}

.btn-icon:hover {
  background-color: #313244;
}

.btn-icon svg {
  width: 22px;
  height: 22px;
}

.btn-main {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  background-color: #b4befe;
  color: #11111b !important;
  box-shadow: 0 8px 20px rgba(180, 190, 254, 0.2);
}

.btn-main:hover {
  box-shadow: 0 12px 28px rgba(180, 190, 254, 0.35);
}

.btn-main svg {
  width: 34px;
  height: 34px;
}

.disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.disabled:hover {
  background-color: #181825;
  color: #a6adc8;
}
</style>
