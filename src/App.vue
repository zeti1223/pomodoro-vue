<script setup>
import { ref, computed, onUnmounted } from "vue";

const DEFAULT_WORK_TIME = 25;
const DEFAULT_BREAK_TIME = 5;

const workTime = ref(DEFAULT_WORK_TIME);
const breakTime = ref(DEFAULT_BREAK_TIME);
const timeLeft = ref(workTime.value * 60);
const isRunning = ref(false);
const isWorkMode = ref(true);
const showSettings = ref(false);
const showNotification = ref(false);
const notificationMessage = ref("");

let timerInterval = null;

const formattedTime = computed(() => {
  const minutes = Math.floor(timeLeft.value / 60);
  const seconds = timeLeft.value % 60;
  return `${minutes.toString().padStart(2, "0")}:${seconds.toString().padStart(2, "0")}`;
});

const currentTotalTime = computed(() => {
  return isWorkMode.value ? workTime.value * 60 : breakTime.value * 60;
});

const progressPercent = computed(() => {
  return (timeLeft.value / currentTotalTime.value) * 100;
});

const modeLabel = computed(() => {
  return isWorkMode.value ? "Work" : "Break";
});

const startTimer = () => {
  if (isRunning.value) return;
  isRunning.value = true;
  timerInterval = setInterval(() => {
    if (timeLeft.value > 0) {
      timeLeft.value--;
    } else {
      handleTimerComplete();
    }
  }, 1000);
};

const handleTimerComplete = () => {
  pauseTimer();
  const message = isWorkMode.value
    ? "Work session complete! Time for a break."
    : "Break is over! Ready to work?";
  notificationMessage.value = message;
  showNotification.value = true;

  if ("Notification" in window && Notification.permission === "granted") {
    new Notification("Pomodoro Timer", { body: message });
  }
};

const pauseTimer = () => {
  isRunning.value = false;
  clearInterval(timerInterval);
};

const resetTimer = () => {
  pauseTimer();
  timeLeft.value = currentTotalTime.value;
};

const switchMode = () => {
  pauseTimer();
  isWorkMode.value = !isWorkMode.value;
  timeLeft.value = currentTotalTime.value;
};

const saveSettings = () => {
  pauseTimer();
  timeLeft.value = currentTotalTime.value;
  showSettings.value = false;
};

const closeNotification = () => {
  showNotification.value = false;
  switchMode();
};

// Request notification permission on mount
if ("Notification" in window && Notification.permission === "default") {
  Notification.requestPermission();
}

onUnmounted(() => {
  clearInterval(timerInterval);
});
</script>

<template>
  <div class="bg-scene" aria-hidden="true">
    <div class="orb orb-a"></div>
    <div class="orb orb-b"></div>
    <div class="orb orb-c"></div>
  </div>

  <main class="app-wrapper">
    <div class="pomodoro-card" :class="{ 'break-mode': !isWorkMode }">
      <header>
        <h1>{{ modeLabel }}</h1>
      </header>

      <div class="timer-circle">
        <svg class="progress-ring" width="300" height="300">
          <defs>
            <linearGradient id="workGradient" x1="0%" y1="0%" x2="100%" y2="100%">
              <stop offset="0%" stop-color="#b4befe" />
              <stop offset="100%" stop-color="#cba6f7" />
            </linearGradient>
            <linearGradient id="breakGradient" x1="0%" y1="0%" x2="100%" y2="100%">
              <stop offset="0%" stop-color="#a6e3a1" />
              <stop offset="100%" stop-color="#89dceb" />
            </linearGradient>
          </defs>
          <circle
            class="progress-ring_bg"
            stroke-width="8"
            fill="transparent"
            r="140"
            cx="150"
            cy="150"
          />
          <circle
            class="progress-ring_circle"
            :class="{ 'break-mode': !isWorkMode }"
            stroke-width="8"
            fill="transparent"
            r="140"
            cx="150"
            cy="150"
            :style="{ strokeDashoffset: 880 - (880 * progressPercent) / 100 }"
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
          <span class="btn-main_sheen" aria-hidden="true"></span>
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

        <button
          class="btn btn-icon"
          @click="showSettings = true"
          title="Settings"
        >
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

    <div
      v-if="showSettings"
      class="modal-overlay"
      @click.self="showSettings = false"
    >
      <div class="modal">
        <h2>Settings</h2>
        <div class="setting-item">
          <label>Work Time (minutes)</label>
          <input
            type="number"
            v-model.number="workTime"
            min="1"
            max="60"
            class="setting-input"
          />
        </div>
        <div class="setting-item">
          <label>Break Time (minutes)</label>
          <input
            type="number"
            v-model.number="breakTime"
            min="1"
            max="30"
            class="setting-input"
          />
        </div>
        <div class="modal-actions">
          <button class="btn btn-secondary" @click="showSettings = false">
            Cancel
          </button>
          <button class="btn btn-primary" @click="saveSettings">Save</button>
        </div>
      </div>
    </div>

    <!-- Notification Modal -->
    <div v-if="showNotification" class="modal-overlay">
      <div class="modal notification-modal">
        <h2>Time's Up!</h2>
        <p>{{ notificationMessage }}</p>
        <button class="btn btn-primary" @click="closeNotification">
          Continue
        </button>
      </div>
    </div>
  </main>
</template>

<style>
body {
  margin: 0;
  background: radial-gradient(ellipse at center, #14141f 0%, #08080e 70%);
  overscroll-behavior: none;
}
</style>

<style scoped>
/* ---------- Ambient background ---------- */
.bg-scene {
  position: fixed;
  inset: 0;
  overflow: hidden;
  z-index: 0;
}

.orb {
  position: absolute;
  border-radius: 50%;
  filter: blur(90px);
  opacity: 0.5;
  will-change: transform;
}

.orb-a {
  width: 480px;
  height: 480px;
  background: #b4befe;
  top: -140px;
  left: -120px;
  animation: float-a 24s ease-in-out infinite;
}

.orb-b {
  width: 420px;
  height: 420px;
  background: #cba6f7;
  bottom: -140px;
  right: -100px;
  animation: float-b 28s ease-in-out infinite;
}

.orb-c {
  width: 360px;
  height: 360px;
  background: #89dceb;
  top: 45%;
  left: 62%;
  opacity: 0.35;
  animation: float-c 32s ease-in-out infinite;
}

@keyframes float-a {
  0%, 100% { transform: translate(0, 0) scale(1); }
  50% { transform: translate(70px, 60px) scale(1.08); }
}

@keyframes float-b {
  0%, 100% { transform: translate(0, 0) scale(1); }
  50% { transform: translate(-60px, -50px) scale(1.06); }
}

@keyframes float-c {
  0%, 100% { transform: translate(0, 0); }
  50% { transform: translate(-40px, 40px); }
}

@media (prefers-reduced-motion: reduce) {
  .orb { animation: none; }
}

/* ---------- Layout ---------- */
.app-wrapper {
  position: relative;
  z-index: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 1.5rem;
  box-sizing: border-box;
  font-family:
    system-ui,
    -apple-system,
    sans-serif;
  color: #eef0fb;
}

/* ---------- Glass card ---------- */
.pomodoro-card {
  position: relative;
  background: linear-gradient(160deg, rgba(255, 255, 255, 0.16), rgba(255, 255, 255, 0.04) 65%);
  backdrop-filter: blur(28px) saturate(170%);
  -webkit-backdrop-filter: blur(28px) saturate(170%);
  padding: 3rem 2.5rem;
  border-radius: 36px;
  border: 1px solid rgba(255, 255, 255, 0.28);
  box-shadow:
    0 30px 60px -15px rgba(0, 0, 0, 0.55),
    inset 0 1px 1px rgba(255, 255, 255, 0.4),
    inset 0 20px 40px -30px rgba(180, 190, 254, 0.25);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2.5rem;
  width: 100%;
  max-width: 420px;
  overflow: hidden;
  transition: box-shadow 0.6s ease, border-color 0.6s ease;
}

.pomodoro-card::before {
  content: "";
  position: absolute;
  top: -50%;
  left: -20%;
  width: 140%;
  height: 90%;
  background: radial-gradient(ellipse at top left, rgba(255, 255, 255, 0.35), transparent 60%);
  transform: rotate(-8deg);
  pointer-events: none;
}

.pomodoro-card.break-mode {
  box-shadow:
    0 30px 60px -15px rgba(0, 0, 0, 0.55),
    inset 0 1px 1px rgba(255, 255, 255, 0.4),
    inset 0 20px 40px -30px rgba(166, 227, 161, 0.25);
  border-color: rgba(166, 227, 161, 0.28);
}

header {
  position: relative;
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
  text-shadow: 0 0 24px rgba(180, 190, 254, 0.45);
  transition: color 0.6s ease, text-shadow 0.6s ease;
}

.pomodoro-card.break-mode h1 {
  color: #a6e3a1;
  text-shadow: 0 0 24px rgba(166, 227, 161, 0.45);
}

/* ---------- Timer ring ---------- */
.timer-circle {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
}

.progress-ring {
  transform: rotate(-90deg);
  overflow: visible;
}

.progress-ring_bg {
  stroke: rgba(255, 255, 255, 0.12);
}

.progress-ring_circle {
  stroke: url(#workGradient);
  stroke-dasharray: 880; /* 2 * PI * R (140) */
  transition: stroke-dashoffset 1s linear, filter 0.6s ease;
  stroke-linecap: round;
  filter: drop-shadow(0 0 10px rgba(180, 190, 254, 0.55));
}

.progress-ring_circle.break-mode {
  stroke: url(#breakGradient);
  filter: drop-shadow(0 0 10px rgba(166, 227, 161, 0.55));
}

.time-display {
  position: absolute;
  font-size: 4.75rem;
  font-weight: 700;
  font-variant-numeric: tabular-nums;
  color: #f4f5fc;
  text-shadow: 0 0 30px rgba(180, 190, 254, 0.35);
  transition: text-shadow 0.6s ease;
}

.pomodoro-card.break-mode .time-display {
  text-shadow: 0 0 30px rgba(166, 227, 161, 0.35);
}

/* ---------- Controls ---------- */
.controls {
  position: relative;
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
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  color: #d4d8f0;
}

.btn:hover {
  color: #ffffff;
}

.btn:active {
  transform: scale(0.92);
}

.btn:focus-visible {
  outline: 2px solid #b4befe;
  outline-offset: 3px;
}

.btn-icon {
  width: 52px;
  height: 52px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(14px) saturate(150%);
  -webkit-backdrop-filter: blur(14px) saturate(150%);
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: inset 0 1px 1px rgba(255, 255, 255, 0.3), 0 6px 16px rgba(0, 0, 0, 0.25);
}

.btn-icon:hover {
  background: rgba(255, 255, 255, 0.16);
  box-shadow: inset 0 1px 1px rgba(255, 255, 255, 0.4), 0 8px 20px rgba(0, 0, 0, 0.3);
}

.btn-icon svg {
  width: 22px;
  height: 22px;
}

.btn-main {
  position: relative;
  width: 80px;
  height: 80px;
  border-radius: 50%;
  background: linear-gradient(160deg, rgba(180, 190, 254, 0.92), rgba(203, 166, 247, 0.78));
  backdrop-filter: blur(14px) saturate(180%);
  -webkit-backdrop-filter: blur(14px) saturate(180%);
  border: 1px solid rgba(255, 255, 255, 0.45);
  color: #14141f !important;
  box-shadow: inset 0 1px 2px rgba(255, 255, 255, 0.65), 0 14px 30px rgba(180, 190, 254, 0.35);
  overflow: hidden;
}

.btn-main:hover {
  box-shadow: inset 0 1px 2px rgba(255, 255, 255, 0.75), 0 18px 38px rgba(180, 190, 254, 0.45);
}

.btn-main svg {
  position: relative;
  z-index: 1;
  width: 34px;
  height: 34px;
}

.btn-main_sheen {
  position: absolute;
  inset: -50%;
  background: conic-gradient(
    from 0deg,
    rgba(255, 255, 255, 0.55),
    transparent 25%,
    transparent 65%,
    rgba(255, 255, 255, 0.55)
  );
  mix-blend-mode: overlay;
  animation: liquid-spin 9s linear infinite;
}

@keyframes liquid-spin {
  to { transform: rotate(360deg); }
}

@media (prefers-reduced-motion: reduce) {
  .btn-main_sheen { animation: none; }
}

.btn-primary {
  background: linear-gradient(160deg, rgba(180, 190, 254, 0.95), rgba(203, 166, 247, 0.8));
  border: 1px solid rgba(255, 255, 255, 0.4);
  color: #14141f;
  padding: 0.75rem 1.5rem;
  border-radius: 14px;
  font-weight: 600;
  box-shadow: inset 0 1px 1px rgba(255, 255, 255, 0.5), 0 8px 18px rgba(180, 190, 254, 0.3);
}

.btn-primary:hover {
  box-shadow: inset 0 1px 1px rgba(255, 255, 255, 0.6), 0 10px 22px rgba(203, 166, 247, 0.4);
}

.btn-secondary {
  background: rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(10px) saturate(150%);
  -webkit-backdrop-filter: blur(10px) saturate(150%);
  border: 1px solid rgba(255, 255, 255, 0.2);
  color: #eef0fb;
  padding: 0.75rem 1.5rem;
  border-radius: 14px;
  font-weight: 600;
}

.btn-secondary:hover {
  background: rgba(255, 255, 255, 0.14);
}

/* ---------- Modals ---------- */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(6, 6, 12, 0.55);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  padding: 1.5rem;
  box-sizing: border-box;
}

.modal {
  position: relative;
  background: linear-gradient(160deg, rgba(255, 255, 255, 0.16), rgba(255, 255, 255, 0.05) 65%);
  backdrop-filter: blur(28px) saturate(170%);
  -webkit-backdrop-filter: blur(28px) saturate(170%);
  padding: 2rem;
  border-radius: 28px;
  border: 1px solid rgba(255, 255, 255, 0.26);
  width: 100%;
  max-width: 400px;
  box-shadow:
    0 30px 60px -15px rgba(0, 0, 0, 0.55),
    inset 0 1px 1px rgba(255, 255, 255, 0.4);
  overflow: hidden;
}

.modal::before {
  content: "";
  position: absolute;
  top: -50%;
  left: -20%;
  width: 140%;
  height: 90%;
  background: radial-gradient(ellipse at top left, rgba(255, 255, 255, 0.3), transparent 60%);
  transform: rotate(-8deg);
  pointer-events: none;
}

.modal h2 {
  position: relative;
  margin: 0 0 1.5rem 0;
  font-size: 1.5rem;
  color: #f4f5fc;
}

.setting-item {
  position: relative;
  margin-bottom: 1.5rem;
}

.setting-item label {
  display: block;
  margin-bottom: 0.5rem;
  color: #c3c8e6;
  font-size: 0.9rem;
}

.setting-input {
  width: 100%;
  padding: 0.75rem;
  background: rgba(255, 255, 255, 0.07);
  backdrop-filter: blur(6px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 14px;
  color: #f4f5fc;
  font-size: 1rem;
  font-family: inherit;
  box-sizing: border-box;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

.setting-input:focus {
  outline: none;
  border-color: #b4befe;
  box-shadow: 0 0 0 3px rgba(180, 190, 254, 0.25);
}

.modal-actions {
  position: relative;
  display: flex;
  gap: 1rem;
  justify-content: flex-end;
  margin-top: 2rem;
}

.notification-modal {
  text-align: center;
  max-width: 320px;
}

.notification-modal p {
  position: relative;
  color: #c3c8e6;
  margin-bottom: 1.5rem;
  line-height: 1.5;
}
</style>
