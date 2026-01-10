<template>
  <TransitionGroup name="ghost">
    <div
      v-for="ghost in visibleGhosts"
      :key="ghost.id"
      class="ghost-popup"
      :style="ghost.style"
      :class="'depth-' + ghost.depth"
    >
      <p>{{ ghost.message }}</p>
    </div>
  </TransitionGroup>
</template>

<script setup>
import { ref, watch, onUnmounted } from 'vue'

const props = defineProps({
  active: { type: Boolean, default: false },
  messages: {
    type: Array,
    default: () => [
      "Един по-малко...",
      "Системата помни.",
      "Защо се отказваш?",
      "Мълчанието убива.",
      "Те печелят.",
      "Пасивност = Съучастие",
      "Гласът ти изчезва...",
      "Без теб няма демокрация.",
      "Вашите данни се изпращат...",
      "Знаем къде живееш.",
      "Няма връщане назад.",
      "Избори без теб = избор срещу теб",
    ]
  },
  interval: { type: Number, default: 2000 },
  maxVisible: { type: Number, default: 3 },
  speakMessages: { type: Boolean, default: false }
})

// TTS за ghost съобщения
function speakText(text) {
  if (props.speakMessages && 'speechSynthesis' in window) {
    const utterance = new SpeechSynthesisUtterance(text)
    utterance.lang = 'bg-BG'
    utterance.rate = 0.85
    utterance.pitch = 0.6
    utterance.volume = 0.8
    speechSynthesis.speak(utterance)
  }
}

const visibleGhosts = ref([])
let ghostId = 0
let intervalId = null

function spawnGhost() {
  if (visibleGhosts.value.length >= props.maxVisible) return

  const depth = Math.floor(Math.random() * 3) + 1 // 1, 2, or 3
  const message = props.messages[Math.floor(Math.random() * props.messages.length)]
  const ghost = {
    id: ghostId++,
    message,
    depth,
    style: {
      top: `${Math.random() * 50 + 20}%`,
      left: `${Math.random() * 50 + 20}%`,
      '--start-x': `${Math.random() * 100 - 50}px`,
      '--start-z': `${Math.random() * 200 - 100}px`,
      animationDuration: `${2 + depth}s`
    }
  }

  visibleGhosts.value.push(ghost)

  // Прочети съобщението
  speakText(message)

  setTimeout(() => {
    const idx = visibleGhosts.value.findIndex(g => g.id === ghost.id)
    if (idx !== -1) visibleGhosts.value.splice(idx, 1)
  }, 2000)
}

watch(() => props.active, (active) => {
  if (active) {
    spawnGhost()
    intervalId = setInterval(spawnGhost, props.interval)
  } else {
    if (intervalId) clearInterval(intervalId)
    visibleGhosts.value = []
  }
}, { immediate: true })

onUnmounted(() => {
  if (intervalId) clearInterval(intervalId)
})
</script>

<style scoped>
.ghost-popup {
  position: fixed;
  padding: 0.8rem 1.2rem;
  background: rgba(139, 0, 0, 0.9);
  color: #fff;
  border-radius: 8px;
  z-index: 900;
  box-shadow: 0 0 20px rgba(255, 0, 0, 0.4);
  animation: ghost-3d-float 3s ease-in-out infinite;
  transform-style: preserve-3d;
  perspective: 1000px;
}

/* Depth layers - closer = bigger, faster */
.depth-1 {
  font-size: 1.1rem;
  z-index: 903;
  filter: blur(0px);
  animation: ghost-3d-close 2s ease-in-out infinite;
}

.depth-2 {
  font-size: 0.9rem;
  z-index: 902;
  filter: blur(0.5px);
  opacity: 0.85;
  animation: ghost-3d-mid 3s ease-in-out infinite;
}

.depth-3 {
  font-size: 0.75rem;
  z-index: 901;
  filter: blur(1px);
  opacity: 0.7;
  animation: ghost-3d-far 4s ease-in-out infinite;
}

@keyframes ghost-3d-close {
  0%, 100% {
    transform: translateY(0) translateX(0) translateZ(0) rotateX(0deg);
  }
  25% {
    transform: translateY(-15px) translateX(10px) translateZ(30px) rotateX(-5deg);
  }
  50% {
    transform: translateY(-5px) translateX(-15px) translateZ(50px) rotateX(5deg);
  }
  75% {
    transform: translateY(-20px) translateX(5px) translateZ(20px) rotateX(-3deg);
  }
}

@keyframes ghost-3d-mid {
  0%, 100% {
    transform: translateY(0) translateX(0) scale(0.9);
  }
  33% {
    transform: translateY(-10px) translateX(-20px) scale(0.95);
  }
  66% {
    transform: translateY(-15px) translateX(15px) scale(0.85);
  }
}

@keyframes ghost-3d-far {
  0%, 100% {
    transform: translateY(0) translateX(0) scale(0.75);
  }
  50% {
    transform: translateY(-8px) translateX(10px) scale(0.8);
  }
}

.ghost-enter-active { animation: ghost-appear 0.3s ease-out; }
.ghost-leave-active { animation: ghost-disappear 0.4s ease-in; }

@keyframes ghost-appear {
  0% { opacity: 0; transform: scale(0.3) translateZ(-100px); }
  100% { opacity: 1; transform: scale(1) translateZ(0); }
}

@keyframes ghost-disappear {
  0% { opacity: 1; transform: scale(1) translateZ(0); }
  100% { opacity: 0; transform: scale(1.5) translateZ(100px); }
}
</style>
