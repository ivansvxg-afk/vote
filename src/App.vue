<template>
  <div class="app-container" :class="{ shaking: effects.screenShake }">
    <!-- EMBEDDED VOTING SITE -->
    <iframe
      class="voting-frame"
      :class="{ 'glitch-mode': effects.glitchText }"
      src="https://azglasuvam.net/#next"
    ></iframe>

    <!-- NEW EFFECTS -->
    <Vignette :intensity="effects.vignette" color="#000000" />
    <HeartbeatPulse :active="effects.heartbeat" :speed="effects.heartbeatSpeed" />
    <ChromaticAberration :intensity="effects.chromatic" />
    <FogLayer :active="effects.fog" />
    <CursorTrail v-if="effects.cursorTrail" />
    <ScreenCrack :active="effects.screenCrack" />

    <!-- DATA SCAN EFFECT -->
    <DataScan :active="effects.dataScan" :intensity="effects.scanIntensity" />

    <!-- 2D EFFECTS OVERLAY -->
    <StaticNoise :intensity="effects.staticNoise" />
    <BloodDrips :active="effects.bloodDrips" :count="8" />
    <GhostMessages :active="effects.ghostMessages" :interval="1500" :max-visible="4" :speak-messages="false" />

    <!-- FINAL QUESTION -->
    <FinalQuestion
      :show="showQuestion"
      @yes="handleYes"
      @no="handleNo"
    />

    <!-- SUCCESS -->
    <SuccessOverlay
      :show="showSuccess"
      @close="handleSuccessClose"
    />

    <!-- SURVEY -->
    <SurveyOverlay
      :show="showSurvey"
      @close="handleSurveyClose"
      @submit="handleSurveySubmit"
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
import DataScan from './components/effects/DataScan.vue'
import Vignette from './components/effects/Vignette.vue'
import HeartbeatPulse from './components/effects/HeartbeatPulse.vue'
import ChromaticAberration from './components/effects/ChromaticAberration.vue'
import FogLayer from './components/effects/FogLayer.vue'
import CursorTrail from './components/effects/CursorTrail.vue'
import ScreenCrack from './components/effects/ScreenCrack.vue'
import FinalQuestion from './components/FinalQuestion.vue'
import SuccessOverlay from './components/SuccessOverlay.vue'
import SurveyOverlay from './components/SurveyOverlay.vue'

// ============ TTS FUNCTIONS ============
function speak(text) {
  if ('speechSynthesis' in window) {
    const utterance = new SpeechSynthesisUtterance(text)
    utterance.lang = 'bg-BG'
    utterance.rate = 0.9
    utterance.pitch = 0.8
    speechSynthesis.speak(utterance)
  }
}

// Voice loop - angry phrases
const angryPhrases = [
  "Твоята пасивност е глас за корупцията.",
  "Докато ти мълчиш, други решават вместо теб.",
  "Докато ти отказваш, на тях им е удобно.",
  "Системата печели когато ти се откажеш.",
  "Мислиш ли че няма значение? Те разчитат на това.",
  "Един глас по-малко. Точно това искат.",
  "Страхът ти е тяхната победа.",
  "Без теб демокрацията умира.",
]

let voiceLoopTimeout = null
let currentPhraseIndex = 0
let vibrationInterval = null

function startVoiceLoop() {
  currentPhraseIndex = 0
  speakNextPhrase()
}

function speakNextPhrase() {
  if ('speechSynthesis' in window) {
    const utterance = new SpeechSynthesisUtterance(angryPhrases[currentPhraseIndex])
    utterance.lang = 'bg-BG'
    utterance.rate = 0.85
    utterance.pitch = 0.7
    utterance.volume = 1.0
    speechSynthesis.speak(utterance)

    currentPhraseIndex = (currentPhraseIndex + 1) % angryPhrases.length
    voiceLoopTimeout = setTimeout(speakNextPhrase, 4000)
  }
}

function stopVoiceLoop() {
  if (voiceLoopTimeout) {
    clearTimeout(voiceLoopTimeout)
    voiceLoopTimeout = null
  }
  if ('speechSynthesis' in window) {
    speechSynthesis.cancel()
  }
}

// ============ VIBRATION LOOP ============
function startVibrationLoop() {
  if (!('vibrate' in navigator)) return

  // Initial VERY strong vibration pattern - alarm style
  navigator.vibrate([500, 100, 500, 100, 500, 100, 800])

  // Continuous aggressive vibration every 1.5 seconds
  vibrationInterval = setInterval(() => {
    navigator.vibrate([400, 80, 400, 80, 400, 80, 600])
  }, 1500)
}

function stopVibrationLoop() {
  if (vibrationInterval) {
    clearInterval(vibrationInterval)
    vibrationInterval = null
  }
  if ('vibrate' in navigator) {
    navigator.vibrate(0) // Stop any ongoing vibration
  }
}

// ============ WHISPERS EFFECT ============
let whispersInterval = null

function startWhispers() {
  if (!('speechSynthesis' in window)) return

  const whisperPhrases = [
    "гласувай...",
    "времето изтича...",
    "те наблюдават...",
    "знаем...",
    "няма бягство...",
  ]

  whispersInterval = setInterval(() => {
    const phrase = whisperPhrases[Math.floor(Math.random() * whisperPhrases.length)]
    const utterance = new SpeechSynthesisUtterance(phrase)
    utterance.lang = 'bg-BG'
    utterance.rate = 0.6
    utterance.pitch = 0.3
    utterance.volume = 0.3
    speechSynthesis.speak(utterance)
  }, 3000)
}

function stopWhispers() {
  if (whispersInterval) {
    clearInterval(whispersInterval)
    whispersInterval = null
  }
}

// ============ BASS DRONE ============
let audioContext = null
let droneOsc = null
let droneGain = null

function startBassDrone() {
  try {
    if (!audioContext) {
      audioContext = new (window.AudioContext || window.webkitAudioContext)()
    }

    droneOsc = audioContext.createOscillator()
    droneGain = audioContext.createGain()

    droneOsc.type = 'sine'
    droneOsc.frequency.setValueAtTime(40, audioContext.currentTime)

    // Slow LFO for movement
    const lfo = audioContext.createOscillator()
    const lfoGain = audioContext.createGain()
    lfo.frequency.value = 0.1
    lfoGain.gain.value = 5
    lfo.connect(lfoGain)
    lfoGain.connect(droneOsc.frequency)
    lfo.start()

    droneGain.gain.setValueAtTime(0, audioContext.currentTime)
    droneGain.gain.linearRampToValueAtTime(0.15, audioContext.currentTime + 2)

    droneOsc.connect(droneGain)
    droneGain.connect(audioContext.destination)
    droneOsc.start()
  } catch (e) {
    console.log('Bass drone error:', e)
  }
}

function stopBassDrone() {
  if (droneGain && audioContext) {
    droneGain.gain.linearRampToValueAtTime(0, audioContext.currentTime + 0.5)
    setTimeout(() => {
      if (droneOsc) {
        droneOsc.stop()
        droneOsc = null
      }
    }, 600)
  }
}

// ============ TIMELINE (30 seconds) ============
const TIMELINE = [
  // Phase 1: Subtle beginning
  { time: 0, action: () => effects.heartbeat = true },
  { time: 1, action: () => effects.staticNoise = 0.02 },
  { time: 2, action: () => effects.vignette = 0.2 },
  { time: 2, action: () => effects.ghostMessages = true },

  // Phase 2: Building tension
  { time: 3, action: () => { effects.staticNoise = 0.04; effects.heartbeatSpeed = 'fast' } },
  { time: 4, action: () => { effects.dataScan = true; effects.chromatic = 0.2 } },
  { time: 5, action: () => { effects.glitchText = true; effects.fog = true } },
  { time: 6, action: () => effects.staticNoise = 0.06 },
  { time: 6, action: () => effects.scanIntensity = 1.5 },

  // Phase 3: Escalation
  { time: 7, action: () => { effects.bloodDrips = true; effects.screenShake = true; setTimeout(() => effects.screenShake = false, 500) } },
  { time: 8, action: () => { effects.scanIntensity = 2; effects.heartbeatSpeed = 'panic' } },
  { time: 8, action: () => { effects.staticNoise = 0.08; effects.vignette = 0.4 } },
  { time: 9, action: () => { effects.chromatic = 0.4 } },

  // Phase 4: Maximum tension
  { time: 10, action: () => { effects.scanIntensity = 2.5; effects.cursorTrail = true } },
  { time: 11, action: () => { effects.staticNoise = 0.1; effects.vignette = 0.5 } },
  { time: 12, action: () => { effects.scanIntensity = 3; effects.chromatic = 0.6 } },
  { time: 13, action: () => effects.screenShake = true },

  // Phase 5: The Question
  { time: 15, action: () => {
    effects.ghostMessages = false
    effects.screenShake = false
    effects.dataScan = false
    showQuestion.value = true
  }},
]

// ============ STATE ============
const effects = reactive({
  staticNoise: 0,
  bloodDrips: false,
  dataScan: false,
  scanIntensity: 1,
  ghostMessages: false,
  glitchText: false,
  vignette: 0,
  heartbeat: false,
  heartbeatSpeed: 'normal',
  chromatic: 0,
  fog: false,
  cursorTrail: false,
  screenCrack: false,
  screenShake: false
})

const showQuestion = ref(false)
const showSuccess = ref(false)
const showSurvey = ref(false)
const sirenAudio = ref(null)

let timeouts = []
let inactivityTimeout = null
let wasTabHidden = false

// ============ TIMELINE CONTROL ============
function startTimeline() {
  TIMELINE.forEach(({ time, action }) => {
    const timeout = setTimeout(action, time * 1000)
    timeouts.push(timeout)
  })
}

function stopAllEffects() {
  timeouts.forEach(clearTimeout)
  timeouts = []

  // Reset all effects
  effects.staticNoise = 0
  effects.bloodDrips = false
  effects.dataScan = false
  effects.scanIntensity = 1
  effects.ghostMessages = false
  effects.glitchText = false
  effects.vignette = 0
  effects.heartbeat = false
  effects.heartbeatSpeed = 'normal'
  effects.chromatic = 0
  effects.fog = false
  effects.cursorTrail = false
  effects.screenCrack = false
  effects.screenShake = false

  // Stop audio and vibration
  stopVoiceLoop()
  stopBassDrone()
  stopVibrationLoop()

  if (sirenAudio.value) {
    sirenAudio.value.pause()
    sirenAudio.value.currentTime = 0
  }

  if ('vibrate' in navigator) {
    navigator.vibrate(0)
  }

  if (audioContext) {
    audioContext.close()
    audioContext = null
  }
}

// ============ INACTIVITY DETECTION ============
function resetInactivityTimer() {
  if (inactivityTimeout) clearTimeout(inactivityTimeout)

  inactivityTimeout = setTimeout(() => {
    if (!showQuestion.value && !showSuccess.value) {
      effects.screenShake = true
      effects.scanIntensity = Math.min(effects.scanIntensity + 0.5, 4)
      setTimeout(() => effects.screenShake = false, 1000)
    }
  }, 8000) // 8 seconds of inactivity
}

// ============ TAB VISIBILITY ============
function handleVisibilityChange() {
  if (document.hidden) {
    wasTabHidden = true
  } else if (wasTabHidden) {
    wasTabHidden = false
    if (!showQuestion.value && !showSuccess.value) {
      effects.chromatic = Math.min(effects.chromatic + 0.2, 1)
      effects.scanIntensity = Math.min(effects.scanIntensity + 0.5, 4)
      effects.screenShake = true
      setTimeout(() => effects.screenShake = false, 500)
    }
  }
}

// ============ EXIT INTENT ============
function handleBeforeUnload(e) {
  if (!showSuccess.value) {
    e.preventDefault()
    e.returnValue = 'Сигурен ли си, че искаш да си тръгнеш? Демокрацията има нужда от теб!'
    return e.returnValue
  }
}

// ============ HANDLERS ============
function handleYes() {
  showQuestion.value = false
  stopAllEffects()
  showSuccess.value = true
}

function handleNo() {
  // Redirect to nightmare page
  window.location.href = '/nightmare.html'
}

function handleSuccessClose() {
  showSuccess.value = false
  showSurvey.value = true
}

function handleSurveyClose() {
  showSurvey.value = false
}

function handleSurveySubmit(result) {
  console.log('Survey submitted:', result)
  // Could send to analytics here
}

// ============ LIFECYCLE ============
onMounted(() => {
  startTimeline()

  // Inactivity detection
  window.addEventListener('mousemove', resetInactivityTimer)
  window.addEventListener('keydown', resetInactivityTimer)
  resetInactivityTimer()

  // Tab visibility
  document.addEventListener('visibilitychange', handleVisibilityChange)

  // Exit intent
  window.addEventListener('beforeunload', handleBeforeUnload)
})

onUnmounted(() => {
  stopAllEffects()

  window.removeEventListener('mousemove', resetInactivityTimer)
  window.removeEventListener('keydown', resetInactivityTimer)
  document.removeEventListener('visibilitychange', handleVisibilityChange)
  window.removeEventListener('beforeunload', handleBeforeUnload)

  if (inactivityTimeout) clearTimeout(inactivityTimeout)
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

.app-container.shaking {
  animation: screen-shake 0.15s infinite;
}

@keyframes screen-shake {
  0%, 100% { transform: translate(0, 0) rotate(0); }
  20% { transform: translate(-3px, 2px) rotate(-0.5deg); }
  40% { transform: translate(3px, -2px) rotate(0.5deg); }
  60% { transform: translate(-2px, -3px) rotate(-0.3deg); }
  80% { transform: translate(2px, 3px) rotate(0.3deg); }
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
