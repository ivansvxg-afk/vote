<template>
  <Transition name="fade">
    <div v-if="show" class="stats-overlay">
      <div class="stats-card">
        <div class="stats-header">
          <h2>Резултати</h2>
          <p class="stats-total">{{ totalVotes }} души отговориха</p>
        </div>

        <!-- Simple Pie Chart -->
        <div class="chart-container">
          <svg viewBox="0 0 200 200" class="pie-chart">
            <!-- Background circle -->
            <circle
              cx="100"
              cy="100"
              r="80"
              fill="none"
              stroke="#333"
              stroke-width="40"
            />
            <!-- Yes segment (green) -->
            <circle
              cx="100"
              cy="100"
              r="80"
              fill="none"
              stroke="#00c853"
              stroke-width="40"
              :stroke-dasharray="yesDashArray"
              stroke-dashoffset="125.6"
              class="pie-segment"
            />
            <!-- No segment (red) -->
            <circle
              cx="100"
              cy="100"
              r="80"
              fill="none"
              stroke="#dd2c00"
              stroke-width="40"
              :stroke-dasharray="noDashArray"
              :stroke-dashoffset="noOffset"
              class="pie-segment"
              style="animation-delay: 0.2s"
            />
          </svg>

          <div class="chart-center">
            <span class="chart-icon">🗳️</span>
          </div>
        </div>

        <!-- Legend -->
        <div class="legend">
          <div class="legend-item yes">
            <span class="legend-color"></span>
            <span class="legend-label">Ще гласувам</span>
            <span class="legend-value">{{ votes.yes }}</span>
          </div>
          <div class="legend-item no">
            <span class="legend-color"></span>
            <span class="legend-label">Няма да гласувам</span>
            <span class="legend-value">{{ votes.no }}</span>
          </div>
        </div>

        <!-- Percentage -->
        <div class="percentage-bar">
          <div class="bar-fill" :style="{ width: yesPercent + '%' }"></div>
          <span class="bar-text">{{ yesPercent }}% избраха да гласуват</span>
        </div>

        <button class="close-btn" @click="$emit('close')">
          Затвори
        </button>
      </div>
    </div>
  </Transition>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'

const props = defineProps({
  show: { type: Boolean, default: false }
})

defineEmits(['close', 'submit'])

const votes = ref({ yes: 0, no: 0 })

const totalVotes = computed(() => votes.value.yes + votes.value.no)

const yesPercent = computed(() => {
  if (totalVotes.value === 0) return 0
  return Math.round((votes.value.yes / totalVotes.value) * 100)
})

const circumference = 2 * Math.PI * 80 // ~502.65

const yesDashArray = computed(() => {
  if (totalVotes.value === 0) return `0 ${circumference}`
  const length = (votes.value.yes / totalVotes.value) * circumference
  return `${length} ${circumference - length}`
})

const noDashArray = computed(() => {
  if (totalVotes.value === 0) return `0 ${circumference}`
  const length = (votes.value.no / totalVotes.value) * circumference
  return `${length} ${circumference - length}`
})

const noOffset = computed(() => {
  if (totalVotes.value === 0) return 125.6
  const yesLength = (votes.value.yes / totalVotes.value) * circumference
  return 125.6 - yesLength
})

function loadVotes() {
  const stored = localStorage.getItem('voteStats')
  if (stored) {
    votes.value = JSON.parse(stored)
  }
}

watch(() => props.show, (newVal) => {
  if (newVal) {
    loadVotes()
  }
})

onMounted(() => {
  loadVotes()
})
</script>

<style scoped>
.stats-overlay {
  position: fixed;
  inset: 0;
  display: grid;
  place-items: center;
  background: rgba(0, 0, 0, 0.9);
  z-index: 2000;
  padding: 1rem;
}

.stats-card {
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
  border-radius: 20px;
  padding: 2rem;
  max-width: 400px;
  width: 100%;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.stats-header {
  text-align: center;
  margin-bottom: 1.5rem;
}

.stats-header h2 {
  color: #fff;
  font-size: 1.5rem;
  margin: 0 0 0.3rem 0;
}

.stats-total {
  color: rgba(255, 255, 255, 0.5);
  font-size: 0.9rem;
  margin: 0;
}

/* Pie Chart */
.chart-container {
  position: relative;
  width: 180px;
  height: 180px;
  margin: 0 auto 1.5rem;
}

.pie-chart {
  width: 100%;
  height: 100%;
  transform: rotate(-90deg);
}

.pie-segment {
  opacity: 0;
  animation: segment-draw 0.8s ease-out forwards;
}

@keyframes segment-draw {
  from {
    opacity: 0;
    stroke-dasharray: 0 502.65;
  }
  to {
    opacity: 1;
  }
}

.chart-center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 90px;
  height: 90px;
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.chart-icon {
  font-size: 2.5rem;
}

/* Legend */
.legend {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
  margin-bottom: 1.5rem;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 0.8rem;
  padding: 0.6rem 1rem;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 10px;
}

.legend-color {
  width: 16px;
  height: 16px;
  border-radius: 4px;
}

.legend-item.yes .legend-color {
  background: #00c853;
  box-shadow: 0 0 10px rgba(0, 200, 83, 0.5);
}

.legend-item.no .legend-color {
  background: #dd2c00;
  box-shadow: 0 0 10px rgba(221, 44, 0, 0.5);
}

.legend-label {
  flex: 1;
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.95rem;
}

.legend-value {
  font-size: 1.2rem;
  font-weight: bold;
  color: #fff;
}

/* Percentage Bar */
.percentage-bar {
  position: relative;
  height: 30px;
  background: rgba(221, 44, 0, 0.3);
  border-radius: 15px;
  overflow: hidden;
  margin-bottom: 1.5rem;
}

.bar-fill {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  background: linear-gradient(90deg, #00c853, #00e676);
  border-radius: 15px;
  transition: width 1s ease-out;
  animation: bar-grow 1s ease-out;
}

@keyframes bar-grow {
  from { width: 0 !important; }
}

.bar-text {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.85rem;
  font-weight: 600;
  color: #fff;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.5);
}

/* Close Button */
.close-btn {
  display: block;
  width: 100%;
  padding: 0.8rem;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 10px;
  color: #fff;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.2s ease;
}

.close-btn:hover {
  background: rgba(255, 255, 255, 0.2);
}

/* Transitions */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* Responsive */
@media (max-width: 400px) {
  .stats-card {
    padding: 1.5rem;
  }

  .chart-container {
    width: 150px;
    height: 150px;
  }

  .chart-center {
    width: 75px;
    height: 75px;
  }

  .chart-icon {
    font-size: 2rem;
  }
}
</style>
