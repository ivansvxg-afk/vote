<template>
  <div v-if="active" class="heartbeat-overlay" :class="{ fast: speed === 'fast', panic: speed === 'panic' }"></div>
</template>

<script setup>
import { watch, onUnmounted } from 'vue'

const props = defineProps({
  active: { type: Boolean, default: false },
  speed: { type: String, default: 'normal' }, // 'normal', 'fast', 'panic'
  withSound: { type: Boolean, default: true }
})

let audioContext = null
let heartbeatInterval = null

function playHeartbeat() {
  if (!props.withSound) return

  try {
    if (!audioContext) {
      audioContext = new (window.AudioContext || window.webkitAudioContext)()
    }

    // First beat (lub)
    const osc1 = audioContext.createOscillator()
    const gain1 = audioContext.createGain()
    osc1.type = 'sine'
    osc1.frequency.setValueAtTime(60, audioContext.currentTime)
    gain1.gain.setValueAtTime(0.4, audioContext.currentTime)
    gain1.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.15)
    osc1.connect(gain1)
    gain1.connect(audioContext.destination)
    osc1.start()
    osc1.stop(audioContext.currentTime + 0.15)

    // Second beat (dub) - slightly delayed
    setTimeout(() => {
      if (!audioContext) return
      const osc2 = audioContext.createOscillator()
      const gain2 = audioContext.createGain()
      osc2.type = 'sine'
      osc2.frequency.setValueAtTime(50, audioContext.currentTime)
      gain2.gain.setValueAtTime(0.3, audioContext.currentTime)
      gain2.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.12)
      osc2.connect(gain2)
      gain2.connect(audioContext.destination)
      osc2.start()
      osc2.stop(audioContext.currentTime + 0.12)
    }, 150)
  } catch (e) {
    console.log('Heartbeat audio error:', e)
  }
}

function getInterval() {
  switch (props.speed) {
    case 'panic': return 400
    case 'fast': return 600
    default: return 1000
  }
}

function startHeartbeat() {
  stopHeartbeat()
  playHeartbeat()
  heartbeatInterval = setInterval(playHeartbeat, getInterval())
}

function stopHeartbeat() {
  if (heartbeatInterval) {
    clearInterval(heartbeatInterval)
    heartbeatInterval = null
  }
}

watch(() => props.active, (active) => {
  if (active) {
    startHeartbeat()
  } else {
    stopHeartbeat()
  }
}, { immediate: true })

watch(() => props.speed, () => {
  if (props.active) {
    startHeartbeat()
  }
})

onUnmounted(() => {
  stopHeartbeat()
  if (audioContext) {
    audioContext.close()
    audioContext = null
  }
})
</script>

<style scoped>
.heartbeat-overlay {
  position: fixed;
  inset: 0;
  z-index: 90;
  pointer-events: none;
  background: radial-gradient(ellipse at center, rgba(255, 0, 0, 0) 0%, rgba(139, 0, 0, 0.3) 100%);
  animation: heartbeat-pulse 1s ease-in-out infinite;
}

.heartbeat-overlay.fast {
  animation-duration: 0.6s;
}

.heartbeat-overlay.panic {
  animation-duration: 0.4s;
  background: radial-gradient(ellipse at center, rgba(255, 0, 0, 0) 0%, rgba(200, 0, 0, 0.5) 100%);
}

@keyframes heartbeat-pulse {
  0%, 100% {
    opacity: 0.3;
    transform: scale(1);
  }
  15% {
    opacity: 1;
    transform: scale(1.02);
  }
  30% {
    opacity: 0.5;
    transform: scale(1);
  }
  45% {
    opacity: 0.9;
    transform: scale(1.01);
  }
  60% {
    opacity: 0.3;
    transform: scale(1);
  }
}
</style>
