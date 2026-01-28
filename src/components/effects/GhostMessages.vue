<template>
  <TransitionGroup
    name="ghost"
    @enter="onGhostEnter"
    @leave="onGhostLeave"
    :css="false"
  >
    <div
      v-for="ghost in visibleGhosts"
      :key="ghost.id"
      class="ghost-popup"
      :style="ghost.style"
      :class="'depth-' + ghost.depth"
      :ref="el => setGhostRef(ghost.id, el)"
    >
      <p class="animated-text">
        <span
          v-for="(char, index) in ghost.message.split('')"
          :key="index"
          class="char"
        >{{ char === ' ' ? '\u00A0' : char }}</span>
      </p>
    </div>
  </TransitionGroup>
</template>

<script setup>
import { ref, watch, onUnmounted } from 'vue'
import gsap from 'gsap'

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

// Store refs to ghost elements
const ghostRefs = ref({})

function setGhostRef(id, el) {
  if (el) {
    ghostRefs.value[id] = el
  } else {
    delete ghostRefs.value[id]
  }
}

// GSAP animation for ghost entrance
function onGhostEnter(el, done) {
  const chars = el.querySelectorAll('.char')
  const depth = el.classList.contains('depth-1') ? 1 :
                el.classList.contains('depth-2') ? 2 : 3

  // Initial state for the container
  gsap.set(el, {
    opacity: 0,
    scale: 0.3,
    z: -100,
    rotationX: 15,
    transformPerspective: 1000,
  })

  // Initial state for all characters
  gsap.set(chars, {
    opacity: 0,
    y: 30,
    rotationX: -90,
    filter: 'blur(8px)',
    transformOrigin: 'center bottom',
  })

  // Create master timeline
  const tl = gsap.timeline({
    onComplete: () => {
      startFloatingAnimation(el, depth)
      done()
    }
  })

  // Container entrance animation
  tl.to(el, {
    opacity: 1,
    scale: 1,
    z: 0,
    rotationX: 0,
    duration: 0.4,
    ease: 'power3.out',
  })

  // Staggered character reveal with blur and 3D rotation
  tl.to(chars, {
    opacity: 1,
    y: 0,
    rotationX: 0,
    filter: 'blur(0px)',
    duration: 0.6,
    stagger: {
      each: 0.03,
      from: 'start',
      ease: 'power2.out',
    },
    ease: 'power3.out',
  }, '-=0.2')

  // Add subtle glow pulse after text appears
  tl.to(el, {
    boxShadow: '0 0 40px rgba(255, 0, 0, 0.6), 0 0 60px rgba(139, 0, 0, 0.4)',
    duration: 0.3,
    ease: 'power2.out',
  }, '-=0.3')

  tl.to(el, {
    boxShadow: '0 0 20px rgba(255, 0, 0, 0.4)',
    duration: 0.4,
    ease: 'power2.inOut',
  })
}

// GSAP animation for ghost exit
function onGhostLeave(el, done) {
  const chars = el.querySelectorAll('.char')

  // Kill any running animations on this element
  gsap.killTweensOf(el)
  gsap.killTweensOf(chars)

  const tl = gsap.timeline({ onComplete: done })

  // Characters scatter and fade with glitch effect
  tl.to(chars, {
    opacity: 0,
    y: () => gsap.utils.random(-30, 30),
    x: () => gsap.utils.random(-20, 20),
    rotationX: () => gsap.utils.random(-45, 45),
    rotationY: () => gsap.utils.random(-30, 30),
    filter: 'blur(6px)',
    duration: 0.4,
    stagger: {
      each: 0.02,
      from: 'random',
    },
    ease: 'power2.in',
  })

  // Container exit with scale and fade
  tl.to(el, {
    opacity: 0,
    scale: 1.3,
    z: 100,
    rotationX: -10,
    filter: 'blur(4px)',
    duration: 0.3,
    ease: 'power2.in',
  }, '-=0.3')
}

// Continuous floating animation based on depth
function startFloatingAnimation(el, depth) {
  // Different animation parameters based on depth
  const params = {
    1: { // Close - more dramatic movement
      yRange: [-15, 15],
      xRange: [-10, 10],
      rotationXRange: [-5, 5],
      duration: 2,
      scale: [1, 1.02],
    },
    2: { // Mid - moderate movement
      yRange: [-10, 10],
      xRange: [-15, 15],
      rotationXRange: [-3, 3],
      duration: 3,
      scale: [0.9, 0.95],
    },
    3: { // Far - subtle movement
      yRange: [-8, 8],
      xRange: [-10, 10],
      rotationXRange: [-2, 2],
      duration: 4,
      scale: [0.75, 0.8],
    }
  }[depth]

  // Create organic floating motion
  gsap.to(el, {
    y: `random(${params.yRange[0]}, ${params.yRange[1]})`,
    x: `random(${params.xRange[0]}, ${params.xRange[1]})`,
    rotationX: `random(${params.rotationXRange[0]}, ${params.rotationXRange[1]})`,
    scale: `random(${params.scale[0]}, ${params.scale[1]})`,
    duration: params.duration,
    ease: 'sine.inOut',
    repeat: -1,
    yoyo: true,
    repeatRefresh: true, // Get new random values each repeat
  })

  // Add subtle glow pulsing
  gsap.to(el, {
    boxShadow: '0 0 30px rgba(255, 0, 0, 0.5), 0 0 50px rgba(139, 0, 0, 0.3)',
    duration: params.duration * 0.6,
    ease: 'sine.inOut',
    repeat: -1,
    yoyo: true,
  })
}

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
    }
  }

  visibleGhosts.value.push(ghost)

  // Прочети съобщението
  speakText(message)

  // Remove ghost after animation duration
  const displayDuration = 2500 + (depth * 500) // Longer display for further ghosts
  setTimeout(() => {
    const idx = visibleGhosts.value.findIndex(g => g.id === ghost.id)
    if (idx !== -1) visibleGhosts.value.splice(idx, 1)
  }, displayDuration)
}

watch(() => props.active, (active) => {
  if (active) {
    spawnGhost()
    intervalId = setInterval(spawnGhost, props.interval)
  } else {
    if (intervalId) clearInterval(intervalId)
    // Kill all GSAP animations on ghost elements
    Object.values(ghostRefs.value).forEach(el => {
      if (el) {
        gsap.killTweensOf(el)
        gsap.killTweensOf(el.querySelectorAll('.char'))
      }
    })
    visibleGhosts.value = []
  }
}, { immediate: true })

onUnmounted(() => {
  if (intervalId) clearInterval(intervalId)
  // Cleanup all GSAP animations
  Object.values(ghostRefs.value).forEach(el => {
    if (el) {
      gsap.killTweensOf(el)
      gsap.killTweensOf(el.querySelectorAll('.char'))
    }
  })
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
  transform-style: preserve-3d;
  perspective: 1000px;
  will-change: transform, opacity, filter;
  backface-visibility: hidden;
}

/* Depth layers - closer = bigger */
.depth-1 {
  font-size: 1.1rem;
  z-index: 903;
  filter: blur(0px);
}

.depth-2 {
  font-size: 0.9rem;
  z-index: 902;
  filter: blur(0.5px);
  opacity: 0.85;
}

.depth-3 {
  font-size: 0.75rem;
  z-index: 901;
  filter: blur(1px);
  opacity: 0.7;
}

/* Letter animation container */
.animated-text {
  display: flex;
  flex-wrap: wrap;
  margin: 0;
  perspective: 600px;
}

.char {
  display: inline-block;
  transform-style: preserve-3d;
  will-change: transform, opacity, filter;
  backface-visibility: hidden;
}
</style>