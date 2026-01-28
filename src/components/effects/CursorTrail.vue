<template>
  <div class="cursor-trail-container">
    <div
      v-for="drop in drops"
      :key="drop.id"
      class="blood-drop"
      :style="drop.style"
    ></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

defineProps({
  active: { type: Boolean, default: false }
})

const drops = ref([])
let dropId = 0
let lastX = 0
let lastY = 0

function onMouseMove(e) {
  const distance = Math.sqrt(
    Math.pow(e.clientX - lastX, 2) + Math.pow(e.clientY - lastY, 2)
  )

  if (distance > 20) {
    lastX = e.clientX
    lastY = e.clientY

    const drop = {
      id: dropId++,
      style: {
        left: `${e.clientX}px`,
        top: `${e.clientY}px`,
        '--fall-distance': `${50 + Math.random() * 100}px`
      }
    }

    drops.value.push(drop)

    // Remove after animation
    setTimeout(() => {
      drops.value = drops.value.filter(d => d.id !== drop.id)
    }, 2000)

    // Limit drops
    if (drops.value.length > 30) {
      drops.value.shift()
    }
  }
}

onMounted(() => {
  window.addEventListener('mousemove', onMouseMove)
})

onUnmounted(() => {
  window.removeEventListener('mousemove', onMouseMove)
})
</script>

<style scoped>
.cursor-trail-container {
  position: fixed;
  inset: 0;
  z-index: 1200;
  pointer-events: none;
  overflow: hidden;
}

.blood-drop {
  position: absolute;
  width: 8px;
  height: 8px;
  background: radial-gradient(circle, #ff0000 0%, #8b0000 70%, transparent 100%);
  border-radius: 50% 50% 50% 50% / 60% 60% 40% 40%;
  transform: translate(-50%, -50%);
  animation: blood-drip 2s ease-in forwards;
  box-shadow: 0 0 6px rgba(255, 0, 0, 0.6);
}

@keyframes blood-drip {
  0% {
    opacity: 1;
    transform: translate(-50%, -50%) scale(1);
  }
  100% {
    opacity: 0;
    transform: translate(-50%, calc(-50% + var(--fall-distance))) scale(0.3);
  }
}
</style>
