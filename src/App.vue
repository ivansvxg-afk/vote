<template>
  <div class="app-container">
    <!-- EMBEDDED VOTING SITE -->
    <iframe
      class="voting-frame"
      :class="{ 'glitch-mode': effects.glitchText }"
      src="https://azglasuvam.net/#next"
    ></iframe>

    <!-- 3D NIGHTMARE SCENE -->
    <NightmareScene
      :intensity="effects.staticNoise"
      :eye-count="effects.eyeCount"
      :blood-particles="effects.bloodDrips"
    />

    <!-- 2D EFFECTS OVERLAY -->
    <StaticNoise :intensity="effects.staticNoise" />
    <BloodDrips :active="effects.bloodDrips" :count="8" />
    <GhostMessages :active="effects.ghostMessages" :interval="1500" :max-visible="4" :speak-messages="true" />

    <!-- FINAL QUESTION -->
    <FinalQuestion
      :show="showQuestion"
      @yes="handleYes"
      @no="handleNo"
    />

    <!-- SUCCESS -->
    <SuccessOverlay
      :show="showSuccess"
      @close="handleClose"
    />

    <!-- AUDIO -->
    <audio ref="sirenAudio" loop preload="auto">
      <source src="/siren.mp3" type="audio/mpeg">
    </audio>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onUnmounted } from 'vue'
import StaticNoise from './components/effects/StaticNoise.vue'
import BloodDrips from './components/effects/BloodDrips.vue'
import GhostMessages from './components/effects/GhostMessages.vue'
import NightmareScene from './components/effects/NightmareScene.vue'
import FinalQuestion from './components/FinalQuestion.vue'
import SuccessOverlay from './components/SuccessOverlay.vue'

// TTS функция
function speak(text) {
  if ('speechSynthesis' in window) {
    const utterance = new SpeechSynthesisUtterance(text)
    utterance.lang = 'bg-BG'
    utterance.rate = 0.9
    utterance.pitch = 0.8
    speechSynthesis.speak(utterance)
  }
}

// Timeline config (време в секунди) - бърза ескалация
const TIMELINE = [
  { time: 2, action: () => effects.staticNoise = 0.03 },
  { time: 4, action: () => effects.ghostMessages = true },
  { time: 5, action: () => effects.staticNoise = 0.05 },
  { time: 7, action: () => effects.eyeCount = 2 },
  { time: 9, action: () => effects.glitchText = true },
  { time: 10, action: () => effects.staticNoise = 0.07 },
  { time: 11, action: () => effects.eyeCount = 4 },
  { time: 13, action: () => effects.bloodDrips = true },
  { time: 14, action: () => effects.eyeCount = 6 },
  { time: 15, action: () => effects.staticNoise = 0.1 },
  { time: 17, action: () => {
    effects.ghostMessages = false  // Спри ghost съобщенията
    if ('speechSynthesis' in window) speechSynthesis.cancel()  // Спри TTS
    showQuestion.value = true
    speak('Ще гласуваш ли?')
  } },
]

// State
const effects = reactive({
  staticNoise: 0,
  bloodDrips: false,
  eyeCount: 0,
  ghostMessages: false,
  glitchText: false
})

const showQuestion = ref(false)
const showSuccess = ref(false)
const sirenAudio = ref(null)

let timeouts = []

function startTimeline() {
  TIMELINE.forEach(({ time, action }) => {
    const timeout = setTimeout(action, time * 1000)
    timeouts.push(timeout)
  })
}

function stopAllEffects() {
  timeouts.forEach(clearTimeout)
  timeouts = []

  effects.staticNoise = 0
  effects.bloodDrips = false
  effects.eyeCount = 0
  effects.ghostMessages = false
  effects.glitchText = false

  // Спри TTS
  if ('speechSynthesis' in window) {
    speechSynthesis.cancel()
  }

  if (sirenAudio.value) {
    sirenAudio.value.pause()
    sirenAudio.value.currentTime = 0
  }
}

function handleYes() {
  showQuestion.value = false
  stopAllEffects()
  showSuccess.value = true
}

function handleNo() {
  // Ескалирай още повече
  effects.staticNoise = 0.15
  speak('Грешен избор!')
  if (sirenAudio.value) sirenAudio.value.play().catch(() => {})
}

function handleClose() {
  showSuccess.value = false
}

onMounted(() => {
  startTimeline()
})

onUnmounted(() => {
  stopAllEffects()
})
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html, body {
  min-height: 100%;
  font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
  background: #0a0a0a;
  color: #eee;
  scroll-behavior: smooth;
}

.app-container {
  min-height: 100vh;
  position: relative;
}

/* EMBEDDED VOTING SITE */
.voting-frame {
  position: fixed;
  inset: 0;
  width: 100%;
  height: 100%;
  border: none;
  z-index: 1;
}

.voting-frame.glitch-mode {
  animation: frame-glitch 0.1s infinite;
}

@keyframes frame-glitch {
  0% { filter: none; }
  25% { filter: hue-rotate(90deg) saturate(1.5); }
  50% { filter: none; transform: translateX(3px); }
  75% { filter: hue-rotate(-90deg) brightness(1.2); }
  100% { filter: none; }
}
</style>
