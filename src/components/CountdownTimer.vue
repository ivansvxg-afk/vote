<template>
  <div class="countdown-container" v-if="daysLeft > 0">
    <div class="countdown-label">До изборите остават:</div>
    <div class="countdown-boxes">
      <div class="countdown-box">
        <span class="countdown-value">{{ countdown.days }}</span>
        <span class="countdown-unit">дни</span>
      </div>
      <div class="countdown-separator">:</div>
      <div class="countdown-box">
        <span class="countdown-value">{{ pad(countdown.hours) }}</span>
        <span class="countdown-unit">часа</span>
      </div>
      <div class="countdown-separator">:</div>
      <div class="countdown-box">
        <span class="countdown-value">{{ pad(countdown.minutes) }}</span>
        <span class="countdown-unit">мин</span>
      </div>
      <div class="countdown-separator">:</div>
      <div class="countdown-box">
        <span class="countdown-value">{{ pad(countdown.seconds) }}</span>
        <span class="countdown-unit">сек</span>
      </div>
    </div>
    <div class="election-date">3 март 2026</div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';

// Фиксирана дата на изборите
const ELECTION_DATE = new Date('2026-03-03T00:00:00');

const countdown = ref({ days: 0, hours: 0, minutes: 0, seconds: 0 });
let intervalId = null;

const daysLeft = computed(() => countdown.value.days);

function pad(num) {
  return String(num).padStart(2, '0');
}

function calculateCountdown() {
  const now = new Date();
  const diff = ELECTION_DATE - now;

  if (diff <= 0) {
    countdown.value = { days: 0, hours: 0, minutes: 0, seconds: 0 };
    return;
  }

  countdown.value = {
    days: Math.floor(diff / (1000 * 60 * 60 * 24)),
    hours: Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60)),
    minutes: Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60)),
    seconds: Math.floor((diff % (1000 * 60)) / 1000)
  };
}

onMounted(() => {
  calculateCountdown();
  intervalId = setInterval(calculateCountdown, 1000);
});

onUnmounted(() => {
  if (intervalId) clearInterval(intervalId);
});
</script>

<style scoped>
.countdown-container {
  margin-bottom: 2rem;
  padding: 1.5rem 2rem;
  background: rgba(0, 200, 180, 0.1);
  border: 2px solid rgba(0, 200, 180, 0.3);
  border-radius: 16px;
  backdrop-filter: blur(10px);
}

.countdown-label {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.7);
  margin-bottom: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 0.1em;
}

.countdown-boxes {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.countdown-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 0.5rem 1rem;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 8px;
  min-width: 70px;
}

.countdown-value {
  font-size: 2.2rem;
  font-weight: bold;
  font-family: 'Courier New', monospace;
  color: #00c8b3;
  text-shadow: 0 0 10px rgba(0, 200, 180, 0.5);
  line-height: 1;
}

.countdown-unit {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.5);
  margin-top: 0.3rem;
  text-transform: uppercase;
}

.countdown-separator {
  font-size: 2rem;
  font-weight: bold;
  color: #00c8b3;
  opacity: 0.5;
  animation: blink-separator 1s infinite;
}

@keyframes blink-separator {
  0%, 100% { opacity: 0.5; }
  50% { opacity: 0.2; }
}

.election-date {
  margin-top: 0.8rem;
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.5);
}

@media (max-width: 500px) {
  .countdown-container {
    padding: 1rem 1.2rem;
  }

  .countdown-box {
    min-width: 55px;
    padding: 0.4rem 0.6rem;
  }

  .countdown-value {
    font-size: 1.6rem;
  }

  .countdown-separator {
    font-size: 1.4rem;
  }
}
</style>