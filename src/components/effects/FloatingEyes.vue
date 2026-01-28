<template>
  <div class="eyes-container">
    <div
      v-for="eye in eyesList"
      :key="eye.id"
      class="eye"
      :style="eye.style"
    >
      <div class="eyeball">
        <div class="veins"></div>
        <div class="iris" :style="getIrisStyle(eye)">
          <div class="pupil"></div>
        </div>
        <div class="reflection"></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  count: { type: Number, default: 0 }
})

const mouse = ref({ x: window.innerWidth / 2, y: window.innerHeight / 2 })
const eyesList = ref([])
let eyeId = 0

function onMouseMove(event) {
  mouse.value = { x: event.clientX, y: event.clientY }
}

function createEye() {
  const size = 50 + Math.random() * 50
  const left = Math.random() * 80 + 10
  const top = Math.random() * 70 + 10
  return {
    id: eyeId++,
    left,
    top,
    size,
    style: {
      left: `${left}%`,
      top: `${top}%`,
      width: `${size}px`,
      height: `${size}px`,
      animationDelay: `${Math.random() * 2}s`,
      animationDuration: `${3 + Math.random() * 2}s`
    }
  }
}

function getIrisStyle(eye) {
  const eyeX = (eye.left / 100) * window.innerWidth
  const eyeY = (eye.top / 100) * window.innerHeight

  const dx = mouse.value.x - eyeX
  const dy = mouse.value.y - eyeY

  // Max movement is 25% of eye size
  const maxMove = eye.size * 0.2
  const angle = Math.atan2(dy, dx)
  const distance = Math.min(Math.sqrt(dx * dx + dy * dy) * 0.03, maxMove)

  const moveX = Math.cos(angle) * distance
  const moveY = Math.sin(angle) * distance

  return {
    transform: `translate(calc(-50% + ${moveX}px), calc(-50% + ${moveY}px))`
  }
}

watch(() => props.count, (newCount) => {
  while (eyesList.value.length < newCount) {
    eyesList.value.push(createEye())
  }
  while (eyesList.value.length > newCount) {
    eyesList.value.pop()
  }
}, { immediate: true })

onMounted(() => {
  window.addEventListener('mousemove', onMouseMove)
})

onUnmounted(() => {
  window.removeEventListener('mousemove', onMouseMove)
})
</script>

<style scoped>
.eyes-container {
  position: fixed;
  inset: 0;
  z-index: 100;
  pointer-events: none;
  overflow: hidden;
}


.eye {
  position: absolute;
  animation: eye-float 4s ease-in-out infinite;
  filter: drop-shadow(0 0 15px rgba(255, 0, 0, 0.5));
}

.eyeball {
  width: 100%;
  height: 100%;
  background: radial-gradient(circle at 35% 35%,
    #ffffff 0%,
    #fff8f8 40%,
    #ffe0e0 70%,
    #ffd0d0 100%
  );
  border-radius: 50%;
  position: relative;
  box-shadow:
    inset -5px -5px 15px rgba(0, 0, 0, 0.1),
    inset 5px 5px 15px rgba(255, 255, 255, 0.8);
  overflow: hidden;
}

.veins {
  position: absolute;
  inset: 0;
  border-radius: 50%;
  background:
    radial-gradient(ellipse 50% 10% at 15% 30%, transparent 40%, #cc3333 50%, transparent 51%),
    radial-gradient(ellipse 10% 40% at 85% 40%, transparent 40%, #bb2222 50%, transparent 51%),
    radial-gradient(ellipse 40% 8% at 20% 70%, transparent 40%, #cc4444 50%, transparent 51%),
    radial-gradient(ellipse 8% 35% at 80% 65%, transparent 40%, #aa3333 50%, transparent 51%),
    radial-gradient(ellipse 35% 6% at 50% 15%, transparent 40%, #bb3333 50%, transparent 51%),
    radial-gradient(ellipse 6% 30% at 10% 50%, transparent 40%, #cc2222 50%, transparent 51%);
  opacity: 0.6;
}

.iris {
  position: absolute;
  width: 50%;
  height: 50%;
  top: 50%;
  left: 50%;
  background: radial-gradient(circle,
    #000000 0%,
    #1a0000 15%,
    #440000 30%,
    #880000 50%,
    #cc2200 70%,
    #ff4400 85%,
    #ff6600 100%
  );
  border-radius: 50%;
  transition: transform 0.08s ease-out;
  box-shadow:
    0 0 10px rgba(255, 68, 0, 0.8),
    0 0 20px rgba(255, 0, 0, 0.5),
    inset 0 0 10px rgba(0, 0, 0, 0.5);
}

.iris::before {
  content: '';
  position: absolute;
  inset: 10%;
  background: repeating-conic-gradient(
    from 0deg,
    rgba(0, 0, 0, 0.4) 0deg 5deg,
    transparent 5deg 10deg
  );
  border-radius: 50%;
  mix-blend-mode: overlay;
}

.pupil {
  position: absolute;
  width: 20%;
  height: 55%;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: #000;
  border-radius: 40%;
  box-shadow:
    0 0 5px #000,
    0 0 10px rgba(255, 0, 0, 0.5);
}

.pupil::after {
  content: '';
  position: absolute;
  inset: -100%;
  background: radial-gradient(circle, rgba(255, 0, 0, 0.2) 0%, transparent 50%);
}

.reflection {
  position: absolute;
  width: 18%;
  height: 12%;
  top: 22%;
  left: 22%;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 50%;
  transform: rotate(-20deg);
}

.reflection::after {
  content: '';
  position: absolute;
  width: 60%;
  height: 60%;
  top: 150%;
  left: 30%;
  background: rgba(255, 255, 255, 0.4);
  border-radius: 50%;
}

@keyframes eye-float {
  0%, 100% {
    transform: translateY(0) scale(1);
  }
  25% {
    transform: translateY(-12px) scale(1.02);
  }
  50% {
    transform: translateY(-6px) scale(0.98);
  }
  75% {
    transform: translateY(-18px) scale(1.01);
  }
}

.eye:nth-child(odd) {
  animation-name: eye-float-alt;
}

@keyframes eye-float-alt {
  0%, 100% {
    transform: translateY(0) scale(1) rotate(0deg);
  }
  33% {
    transform: translateY(-15px) scale(1.03) rotate(3deg);
  }
  66% {
    transform: translateY(-8px) scale(0.97) rotate(-2deg);
  }
}

/* Random blink */
.eye:nth-child(3n) .eyeball {
  animation: blink 5s ease-in-out infinite;
  animation-delay: 1s;
}

.eye:nth-child(4n+1) .eyeball {
  animation: blink 6s ease-in-out infinite;
  animation-delay: 2.5s;
}

@keyframes blink {
  0%, 92%, 100% { transform: scaleY(1); }
  96% { transform: scaleY(0.05); }
}
</style>
