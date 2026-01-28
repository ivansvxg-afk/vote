<template>
  <div v-if="active" class="crack-container" :class="{ animating: animating }">
    <svg class="crack-svg" viewBox="0 0 100 100" preserveAspectRatio="none">
      <!-- Main crack from center -->
      <path class="crack-line main" d="M50,50 L48,30 L52,15 L49,0" />
      <path class="crack-line main" d="M50,50 L55,35 L53,20 L58,0" />
      <path class="crack-line main" d="M50,50 L45,40 L40,25 L42,0" />

      <!-- Cracks going down -->
      <path class="crack-line main" d="M50,50 L52,70 L48,85 L51,100" />
      <path class="crack-line main" d="M50,50 L46,65 L50,80 L47,100" />

      <!-- Cracks going left -->
      <path class="crack-line main" d="M50,50 L30,48 L15,52 L0,49" />
      <path class="crack-line main" d="M50,50 L35,55 L20,53 L0,58" />

      <!-- Cracks going right -->
      <path class="crack-line main" d="M50,50 L70,52 L85,48 L100,51" />
      <path class="crack-line main" d="M50,50 L65,45 L80,50 L100,45" />

      <!-- Secondary smaller cracks -->
      <path class="crack-line secondary" d="M48,30 L40,28 L35,32" />
      <path class="crack-line secondary" d="M52,15 L58,12 L62,18" />
      <path class="crack-line secondary" d="M30,48 L28,40 L32,35" />
      <path class="crack-line secondary" d="M70,52 L75,60 L80,58" />
      <path class="crack-line secondary" d="M52,70 L60,75 L58,82" />
      <path class="crack-line secondary" d="M46,65 L38,68 L35,75" />
    </svg>

    <!-- Blood seeping through cracks -->
    <div class="blood-seep seep-1"></div>
    <div class="blood-seep seep-2"></div>
    <div class="blood-seep seep-3"></div>

    <!-- Impact point -->
    <div class="impact-point"></div>
  </div>
</template>

<script setup>
import { ref, watch, onUnmounted } from 'vue'

const props = defineProps({
  active: { type: Boolean, default: false }
})

const animating = ref(false)
let audioContext = null

function playCrackSound() {
  try {
    if (!audioContext) {
      audioContext = new (window.AudioContext || window.webkitAudioContext)()
    }

    // Glass breaking noise
    const bufferSize = audioContext.sampleRate * 0.5
    const buffer = audioContext.createBuffer(1, bufferSize, audioContext.sampleRate)
    const data = buffer.getChannelData(0)

    for (let i = 0; i < bufferSize; i++) {
      // High frequency noise for glass
      data[i] = (Math.random() * 2 - 1) * Math.exp(-i / (bufferSize * 0.05))
    }

    const noise = audioContext.createBufferSource()
    noise.buffer = buffer

    // High-pass filter for glass-like sound
    const filter = audioContext.createBiquadFilter()
    filter.type = 'highpass'
    filter.frequency.value = 2000

    const gain = audioContext.createGain()
    gain.gain.setValueAtTime(0.6, audioContext.currentTime)
    gain.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.5)

    noise.connect(filter)
    filter.connect(gain)
    gain.connect(audioContext.destination)

    noise.start()
  } catch (e) {
    console.log('Crack sound error:', e)
  }
}

watch(() => props.active, (active) => {
  if (active) {
    animating.value = true
    playCrackSound()
  }
})

onUnmounted(() => {
  if (audioContext) {
    audioContext.close()
  }
})
</script>

<style scoped>
.crack-container {
  position: fixed;
  inset: 0;
  z-index: 2000;
  pointer-events: none;
}

.crack-svg {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
}

.crack-line {
  fill: none;
  stroke: rgba(255, 255, 255, 0.9);
  stroke-linecap: round;
  stroke-dasharray: 200;
  stroke-dashoffset: 200;
  filter: drop-shadow(0 0 2px rgba(255, 255, 255, 0.8));
}

.crack-line.main {
  stroke-width: 0.3;
}

.crack-line.secondary {
  stroke-width: 0.15;
  opacity: 0.7;
}

.animating .crack-line.main {
  animation: crack-draw 0.3s ease-out forwards;
}

.animating .crack-line.secondary {
  animation: crack-draw 0.3s ease-out 0.15s forwards;
}

@keyframes crack-draw {
  to {
    stroke-dashoffset: 0;
  }
}

.blood-seep {
  position: absolute;
  width: 20px;
  height: 60px;
  background: linear-gradient(to bottom, #8b0000 0%, #ff0000 50%, transparent 100%);
  border-radius: 50%;
  opacity: 0;
  filter: blur(2px);
}

.animating .blood-seep {
  animation: seep-down 3s ease-in 0.5s forwards;
}

.seep-1 {
  top: 50%;
  left: 48%;
}

.seep-2 {
  top: 50%;
  left: 52%;
  animation-delay: 0.8s !important;
}

.seep-3 {
  top: 50%;
  left: 50%;
  animation-delay: 1.2s !important;
}

@keyframes seep-down {
  0% {
    opacity: 0;
    transform: translateY(0) scaleY(0.5);
  }
  20% {
    opacity: 0.8;
  }
  100% {
    opacity: 0.6;
    transform: translateY(200px) scaleY(2);
  }
}

.impact-point {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 30px;
  height: 30px;
  transform: translate(-50%, -50%);
  background: radial-gradient(circle, rgba(255, 255, 255, 0.9) 0%, transparent 70%);
  border-radius: 50%;
  opacity: 0;
}

.animating .impact-point {
  animation: impact-flash 0.2s ease-out forwards;
}

@keyframes impact-flash {
  0% {
    opacity: 1;
    transform: translate(-50%, -50%) scale(0);
  }
  50% {
    opacity: 1;
    transform: translate(-50%, -50%) scale(2);
  }
  100% {
    opacity: 0;
    transform: translate(-50%, -50%) scale(3);
  }
}
</style>
