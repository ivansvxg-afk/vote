<template>
  <Transition name="crash-enter">
    <div v-if="show" class="question-overlay" :class="{ shaking: isShaking }" @click="handleOverlayClick" @touchstart="handleOverlayTouch">
      <!-- Glitch layers -->
      <div class="glitch-layer glitch-1"></div>
      <div class="glitch-layer glitch-2"></div>
      <div class="glitch-layer glitch-3"></div>

      <!-- Flash effect -->
      <div class="flash-overlay" :class="{ active: flashActive }"></div>

      <!-- Scanlines -->
      <div class="scanlines"></div>

      <!-- Static noise -->
      <div class="static-bg"></div>

      <div class="question-popup">
        <!-- Glitching border -->
        <div class="glitch-border"></div>
        <div class="glitch-border delay-1"></div>
        <div class="glitch-border delay-2"></div>

        <!-- Warning stripes -->
        <div class="warning-stripes top"></div>
        <div class="warning-stripes bottom"></div>

        <!-- Countdown Timer -->
        <CountdownTimer />

        <!-- Icon -->
        <div class="icon-container">
          <div class="icon-pulse"></div>
          <div class="icon-pulse delay"></div>
          <div class="warning-icon">⚠</div>
        </div>

        <h2 class="question-title glitch-text" data-text="ЩЕ ГЛАСУВАШ ЛИ?">ЩЕ ГЛАСУВАШ ЛИ?</h2>

        <p class="question-text">
          <span class="blink">ВНИМАНИЕ:</span> Демокрацията е в опасност.<br>
          <span class="highlight">ТВОЯТ ГЛАС РЕШАВА ВСИЧКО.</span>
        </p>

        <div class="question-buttons">
          <button class="btn btn-yes" @click="handleYes">
            <span class="btn-text">ДА, ЩЕ ГЛАСУВАМ!</span>
          </button>
          <button class="btn btn-no" @click="handleNo">
            <span class="btn-text">НЕ</span>
          </button>
        </div>

        <!-- Corner decorations -->
        <div class="corner tl"></div>
        <div class="corner tr"></div>
        <div class="corner bl"></div>
        <div class="corner br"></div>
      </div>
    </div>
  </Transition>
</template>

<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue'
import CountdownTimer from './CountdownTimer.vue'

const props = defineProps({
  show: { type: Boolean, default: false }
})

const emit = defineEmits(['yes', 'no'])

const isShaking = ref(false)
const flashActive = ref(false)
let audioContext = null
let shakeInterval = null

// Create audio context for sound effects
function initAudio() {
  if (!audioContext) {
    audioContext = new (window.AudioContext || window.webkitAudioContext)()
  }
  return audioContext
}

// Generate crash/impact sound
function playCrashSound() {
  try {
    const ctx = initAudio()

    // White noise burst
    const bufferSize = ctx.sampleRate * 0.3
    const buffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate)
    const data = buffer.getChannelData(0)

    for (let i = 0; i < bufferSize; i++) {
      data[i] = (Math.random() * 2 - 1) * Math.exp(-i / (bufferSize * 0.1))
    }

    const noise = ctx.createBufferSource()
    noise.buffer = buffer

    // Low frequency oscillator for impact
    const osc = ctx.createOscillator()
    osc.type = 'sawtooth'
    osc.frequency.setValueAtTime(80, ctx.currentTime)
    osc.frequency.exponentialRampToValueAtTime(20, ctx.currentTime + 0.3)

    // Gain for noise
    const noiseGain = ctx.createGain()
    noiseGain.gain.setValueAtTime(0.5, ctx.currentTime)
    noiseGain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.3)

    // Gain for oscillator
    const oscGain = ctx.createGain()
    oscGain.gain.setValueAtTime(0.4, ctx.currentTime)
    oscGain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.4)

    // Distortion
    const distortion = ctx.createWaveShaper()
    distortion.curve = makeDistortionCurve(400)

    noise.connect(noiseGain)
    noiseGain.connect(distortion)
    osc.connect(oscGain)
    oscGain.connect(distortion)
    distortion.connect(ctx.destination)

    noise.start()
    osc.start()
    osc.stop(ctx.currentTime + 0.4)
  } catch (e) {
    console.log('Audio not available')
  }
}

// Generate alarm/warning sound
function playAlarmSound() {
  try {
    const ctx = initAudio()

    const osc1 = ctx.createOscillator()
    const osc2 = ctx.createOscillator()
    const gain = ctx.createGain()

    osc1.type = 'square'
    osc2.type = 'square'

    // Alternating frequencies for alarm effect
    osc1.frequency.setValueAtTime(800, ctx.currentTime)
    osc2.frequency.setValueAtTime(600, ctx.currentTime)

    gain.gain.setValueAtTime(0.15, ctx.currentTime)

    // Tremolo effect
    const lfo = ctx.createOscillator()
    const lfoGain = ctx.createGain()
    lfo.frequency.value = 8
    lfoGain.gain.value = 0.15

    lfo.connect(lfoGain)
    lfoGain.connect(gain.gain)

    osc1.connect(gain)
    osc2.connect(gain)
    gain.connect(ctx.destination)

    lfo.start()
    osc1.start()
    osc2.start()

    // Stop after 0.5 seconds
    setTimeout(() => {
      gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 0.1)
      setTimeout(() => {
        osc1.stop()
        osc2.stop()
        lfo.stop()
      }, 100)
    }, 500)
  } catch (e) {
    console.log('Audio not available')
  }
}

// Distortion curve
function makeDistortionCurve(amount) {
  const samples = 44100
  const curve = new Float32Array(samples)
  for (let i = 0; i < samples; i++) {
    const x = (i * 2) / samples - 1
    curve[i] = ((Math.PI + amount) * x) / (Math.PI + amount * Math.abs(x))
  }
  return curve
}

// Vibration helper
function vibrate(pattern) {
  if ('vibrate' in navigator) {
    navigator.vibrate(pattern)
  }
}

// BASS RUMBLE - creates a deep bass sound that feels like vibration
function playBassRumble(duration = 1.5, intensity = 0.8) {
  try {
    const ctx = initAudio()

    // Main bass oscillator (very low frequency)
    const bass = ctx.createOscillator()
    bass.type = 'sine'
    bass.frequency.setValueAtTime(30, ctx.currentTime) // Sub-bass

    // Second bass layer
    const bass2 = ctx.createOscillator()
    bass2.type = 'sine'
    bass2.frequency.setValueAtTime(45, ctx.currentTime)

    // LFO for pulsing effect
    const lfo = ctx.createOscillator()
    const lfoGain = ctx.createGain()
    lfo.frequency.value = 8 // Pulse rate
    lfoGain.gain.value = 15
    lfo.connect(lfoGain)
    lfoGain.connect(bass.frequency)

    // Distortion for more impact
    const distortion = ctx.createWaveShaper()
    distortion.curve = makeDistortionCurve(50)

    // Main gain
    const gain = ctx.createGain()
    gain.gain.setValueAtTime(0, ctx.currentTime)
    gain.gain.linearRampToValueAtTime(intensity, ctx.currentTime + 0.05)
    gain.gain.setValueAtTime(intensity, ctx.currentTime + duration - 0.2)
    gain.gain.linearRampToValueAtTime(0, ctx.currentTime + duration)

    // Connect everything
    bass.connect(distortion)
    bass2.connect(distortion)
    distortion.connect(gain)
    gain.connect(ctx.destination)

    // Start
    lfo.start()
    bass.start()
    bass2.start()

    // Stop after duration
    setTimeout(() => {
      bass.stop()
      bass2.stop()
      lfo.stop()
    }, duration * 1000)
  } catch (e) {
    console.log('Bass rumble error:', e)
  }
}

// Short bass hit for interactions
function playBassHit() {
  try {
    const ctx = initAudio()

    const osc = ctx.createOscillator()
    osc.type = 'sine'
    osc.frequency.setValueAtTime(50, ctx.currentTime)
    osc.frequency.exponentialRampToValueAtTime(20, ctx.currentTime + 0.3)

    const gain = ctx.createGain()
    gain.gain.setValueAtTime(0.9, ctx.currentTime)
    gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.3)

    osc.connect(gain)
    gain.connect(ctx.destination)

    osc.start()
    osc.stop(ctx.currentTime + 0.3)
  } catch (e) {}
}

// Handle any touch/click on the overlay - trigger vibration + bass
function handleOverlayClick() {
  vibrate([300, 100, 300])
  playBassHit()
}

function handleOverlayTouch() {
  vibrate([300, 100, 300])
  playBassHit()
}

// Trigger effects when shown
let bassInterval = null

watch(() => props.show, (newVal) => {
  if (newVal) {
    // Flash effect
    flashActive.value = true
    setTimeout(() => flashActive.value = false, 150)

    // Play sounds
    playCrashSound()
    setTimeout(() => playAlarmSound(), 200)

    // BASS RUMBLE - deep vibration-like sound
    playBassRumble(2, 0.9)

    // VIBRATION - aggressive alarm pattern
    vibrate([500, 100, 500, 100, 500, 100, 800])

    // Start shaking with continuous bass
    isShaking.value = true
    shakeInterval = setInterval(() => {
      isShaking.value = !isShaking.value
      setTimeout(() => isShaking.value = true, 50)
      // Vibrate on each shake cycle
      vibrate([200, 100, 200])
    }, 2000)

    // Continuous bass rumble every 3 seconds
    bassInterval = setInterval(() => {
      playBassRumble(1.5, 0.7)
    }, 3000)
  } else {
    isShaking.value = false
    if (shakeInterval) clearInterval(shakeInterval)
    if (bassInterval) clearInterval(bassInterval)
    vibrate(0) // Stop vibration
  }
})

function saveVote(willVote) {
  const votes = JSON.parse(localStorage.getItem('voteStats') || '{"yes":0,"no":0}')
  if (willVote) {
    votes.yes++
  } else {
    votes.no++
  }
  localStorage.setItem('voteStats', JSON.stringify(votes))
}

function handleYes() {
  // Save vote
  saveVote(true)

  // Success sound
  try {
    const ctx = initAudio()
    const osc = ctx.createOscillator()
    const gain = ctx.createGain()
    osc.type = 'sine'
    osc.frequency.setValueAtTime(523, ctx.currentTime)
    osc.frequency.setValueAtTime(659, ctx.currentTime + 0.1)
    osc.frequency.setValueAtTime(784, ctx.currentTime + 0.2)
    gain.gain.setValueAtTime(0.2, ctx.currentTime)
    gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.4)
    osc.connect(gain)
    gain.connect(ctx.destination)
    osc.start()
    osc.stop(ctx.currentTime + 0.4)
  } catch (e) {}

  emit('yes')
}

function handleNo() {
  // Save vote
  saveVote(false)

  // Error/danger sound
  try {
    const ctx = initAudio()
    const osc = ctx.createOscillator()
    const gain = ctx.createGain()
    osc.type = 'sawtooth'
    osc.frequency.setValueAtTime(150, ctx.currentTime)
    osc.frequency.setValueAtTime(100, ctx.currentTime + 0.2)
    gain.gain.setValueAtTime(0.3, ctx.currentTime)
    gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.3)
    osc.connect(gain)
    gain.connect(ctx.destination)
    osc.start()
    osc.stop(ctx.currentTime + 0.3)
  } catch (e) {}

  emit('no')
}

onUnmounted(() => {
  if (shakeInterval) clearInterval(shakeInterval)
  if (bassInterval) clearInterval(bassInterval)
})
</script>

<style scoped>
.question-overlay {
  position: fixed;
  inset: 0;
  display: grid;
  place-items: center;
  background: rgba(0, 0, 0, 0.7);
  z-index: 1000;
  overflow: hidden;
}

.question-overlay.shaking {
  animation: shake 0.1s infinite;
}

@keyframes shake {
  0%, 100% { transform: translate(0, 0) rotate(0deg); }
  25% { transform: translate(-5px, 3px) rotate(-0.5deg); }
  50% { transform: translate(5px, -3px) rotate(0.5deg); }
  75% { transform: translate(-3px, -5px) rotate(-0.3deg); }
}

/* Glitch layers */
.glitch-layer {
  position: absolute;
  inset: 0;
  background: linear-gradient(
    45deg,
    transparent 30%,
    rgba(255, 0, 0, 0.1) 30%,
    rgba(255, 0, 0, 0.1) 32%,
    transparent 32%
  );
  animation: glitch-move 0.3s infinite;
}

.glitch-1 { animation-delay: 0s; }
.glitch-2 { animation-delay: 0.1s; opacity: 0.5; }
.glitch-3 { animation-delay: 0.2s; opacity: 0.3; }

@keyframes glitch-move {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(100%); }
}

/* Flash */
.flash-overlay {
  position: absolute;
  inset: 0;
  background: white;
  opacity: 0;
  pointer-events: none;
  z-index: 100;
}

.flash-overlay.active {
  animation: flash 0.15s ease-out;
}

@keyframes flash {
  0% { opacity: 1; }
  100% { opacity: 0; }
}

/* Scanlines */
.scanlines {
  position: absolute;
  inset: 0;
  background: repeating-linear-gradient(
    0deg,
    rgba(0, 0, 0, 0.2) 0px,
    rgba(0, 0, 0, 0.2) 1px,
    transparent 1px,
    transparent 3px
  );
  pointer-events: none;
  z-index: 50;
}

/* Static noise background */
.static-bg {
  position: absolute;
  inset: 0;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)'/%3E%3C/svg%3E");
  opacity: 0.08;
  animation: static-flicker 0.1s infinite;
}

@keyframes static-flicker {
  0%, 100% { opacity: 0.08; }
  50% { opacity: 0.12; }
}

/* Popup */
.question-popup {
  position: relative;
  width: min(600px, calc(100% - 1rem));
  background: linear-gradient(180deg, #1a0000 0%, #330000 50%, #1a0000 100%);
  color: #fff;
  padding: 2.5rem 2rem;
  text-align: center;
  border: 3px solid #ff0000;
  clip-path: polygon(
    0 10px, 10px 0, calc(100% - 10px) 0, 100% 10px,
    100% calc(100% - 10px), calc(100% - 10px) 100%, 10px 100%, 0 calc(100% - 10px)
  );
}

/* Glitching border effect */
.glitch-border {
  position: absolute;
  inset: -3px;
  border: 3px solid #ff0000;
  clip-path: inherit;
  animation: border-glitch 0.2s infinite;
}

.glitch-border.delay-1 {
  animation-delay: 0.05s;
  border-color: #00ffff;
  opacity: 0.5;
}

.glitch-border.delay-2 {
  animation-delay: 0.1s;
  border-color: #ff00ff;
  opacity: 0.3;
}

@keyframes border-glitch {
  0%, 100% { transform: translate(0, 0); }
  20% { transform: translate(-2px, 1px); }
  40% { transform: translate(2px, -1px); }
  60% { transform: translate(-1px, -2px); }
  80% { transform: translate(1px, 2px); }
}

/* Warning stripes */
.warning-stripes {
  position: absolute;
  left: 0;
  right: 0;
  height: 20px;
  background: repeating-linear-gradient(
    -45deg,
    #ff0000,
    #ff0000 10px,
    #000 10px,
    #000 20px
  );
  animation: stripes-move 0.5s linear infinite;
}

.warning-stripes.top { top: 0; }
.warning-stripes.bottom { bottom: 0; }

@keyframes stripes-move {
  0% { background-position: 0 0; }
  100% { background-position: 28px 0; }
}

/* Icon */
.icon-container {
  position: relative;
  width: 100px;
  height: 100px;
  margin: 1rem auto 1.5rem;
}

.icon-pulse {
  position: absolute;
  inset: 0;
  border: 4px solid #ff0000;
  border-radius: 50%;
  animation: icon-pulse 1s ease-out infinite;
}

.icon-pulse.delay {
  animation-delay: 0.5s;
}

@keyframes icon-pulse {
  0% { transform: scale(1); opacity: 1; }
  100% { transform: scale(2); opacity: 0; }
}

.warning-icon {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 4rem;
  color: #ff0000;
  text-shadow:
    0 0 10px #ff0000,
    0 0 20px #ff0000,
    0 0 40px #ff0000;
  animation: icon-blink 0.5s ease-in-out infinite;
}

@keyframes icon-blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

/* Glitch text effect */
.glitch-text {
  position: relative;
  font-size: clamp(2rem, 6vw, 3rem);
  font-weight: 900;
  text-transform: uppercase;
  color: #fff;
  text-shadow:
    0 0 10px #ff0000,
    0 0 20px #ff0000;
  animation: glitch-text 0.3s infinite;
}

.glitch-text::before,
.glitch-text::after {
  content: attr(data-text);
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
}

.glitch-text::before {
  color: #ff0000;
  animation: glitch-text-1 0.2s infinite;
  clip-path: polygon(0 0, 100% 0, 100% 45%, 0 45%);
}

.glitch-text::after {
  color: #00ffff;
  animation: glitch-text-2 0.2s infinite;
  clip-path: polygon(0 55%, 100% 55%, 100% 100%, 0 100%);
}

@keyframes glitch-text {
  0%, 100% { transform: translate(0); }
  20% { transform: translate(-2px, 2px); }
  40% { transform: translate(2px, -2px); }
  60% { transform: translate(-2px, -2px); }
  80% { transform: translate(2px, 2px); }
}

@keyframes glitch-text-1 {
  0%, 100% { transform: translate(0); }
  50% { transform: translate(3px, 0); }
}

@keyframes glitch-text-2 {
  0%, 100% { transform: translate(0); }
  50% { transform: translate(-3px, 0); }
}

.question-text {
  margin: 1.5rem 0 2rem;
  font-size: 1.1rem;
  color: #ccc;
  line-height: 1.8;
}

.blink {
  color: #ff0000;
  font-weight: bold;
  animation: blink 0.5s infinite;
}

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0; }
}

.highlight {
  color: #ffff00;
  font-weight: bold;
  text-shadow: 0 0 10px #ffff00;
}

/* Buttons */
.question-buttons {
  display: flex;
  gap: 1rem;
  justify-content: center;
  flex-wrap: wrap;
  margin-top: 1rem;
}

.btn {
  position: relative;
  padding: 1rem 2.5rem;
  border: none;
  font-size: 1.2rem;
  font-weight: 900;
  text-transform: uppercase;
  cursor: pointer;
  transition: all 0.2s;
  clip-path: polygon(10px 0, 100% 0, calc(100% - 10px) 100%, 0 100%);
}

.btn-yes {
  background: #00ff00;
  color: #000;
  box-shadow: 0 0 20px rgba(0, 255, 0, 0.5);
}

.btn-yes:hover {
  background: #00ff88;
  transform: scale(1.05);
  box-shadow: 0 0 40px rgba(0, 255, 0, 0.8);
}

.btn-no {
  background: #333;
  color: #666;
  border: 2px solid #666;
}

.btn-no:hover {
  background: #ff0000;
  color: #fff;
  border-color: #ff0000;
  box-shadow: 0 0 30px rgba(255, 0, 0, 0.6);
}

/* Corners */
.corner {
  position: absolute;
  width: 20px;
  height: 20px;
  border: 3px solid #ff0000;
}

.corner.tl { top: 25px; left: 10px; border-right: none; border-bottom: none; }
.corner.tr { top: 25px; right: 10px; border-left: none; border-bottom: none; }
.corner.bl { bottom: 25px; left: 10px; border-right: none; border-top: none; }
.corner.br { bottom: 25px; right: 10px; border-left: none; border-top: none; }

/* Transition */
.crash-enter-enter-active {
  animation: crash-in 0.3s cubic-bezier(0.36, 0.07, 0.19, 0.97);
}

.crash-enter-leave-active {
  animation: crash-out 0.2s ease-in;
}

@keyframes crash-in {
  0% {
    opacity: 0;
    transform: scale(1.5) rotate(5deg);
    filter: blur(20px);
  }
  50% {
    transform: scale(0.9) rotate(-2deg);
    filter: blur(0);
  }
  100% {
    opacity: 1;
    transform: scale(1) rotate(0);
  }
}

@keyframes crash-out {
  0% { opacity: 1; transform: scale(1); }
  100% { opacity: 0; transform: scale(0.5) rotate(10deg); }
}

/* Responsive */
@media (max-width: 500px) {
  .question-popup {
    padding: 2rem 1rem;
  }

  .btn {
    padding: 0.8rem 1.5rem;
    font-size: 1rem;
  }

  .warning-icon {
    font-size: 3rem;
  }
}
</style>
