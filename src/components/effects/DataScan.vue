<template>
  <div v-if="active" class="scan-container">
    <!-- Scanning lines -->
    <div class="scan-line"></div>
    <div class="scan-line delay-1"></div>

    <!-- Corner brackets -->
    <div class="corner-bracket tl"></div>
    <div class="corner-bracket tr"></div>
    <div class="corner-bracket bl"></div>
    <div class="corner-bracket br"></div>

    <!-- Data streams -->
    <div class="data-stream left">
      <div v-for="i in 25" :key="'l'+i" class="data-bit" :style="{ animationDelay: `${i * 0.08}s` }">
        {{ randomHex() }}
      </div>
    </div>
    <div class="data-stream right">
      <div v-for="i in 25" :key="'r'+i" class="data-bit" :style="{ animationDelay: `${i * 0.12}s` }">
        {{ randomHex() }}
      </div>
    </div>

    <!-- Stolen data popups -->
    <TransitionGroup name="popup">
      <div
        v-for="item in stolenData"
        :key="item.id"
        class="stolen-item"
        :style="item.style"
      >
        <div class="stolen-header">
          <span class="stolen-icon">{{ item.icon }}</span>
          <span class="stolen-label">{{ item.label }}</span>
        </div>
        <div class="stolen-value">{{ item.value }}</div>
        <div class="stolen-status">
          <span class="status-dot"></span>
          {{ item.status }}
        </div>
      </div>
    </TransitionGroup>

    <!-- Terminal output -->
    <div class="terminal" v-if="showTerminal">
      <div class="terminal-header">
        <span class="terminal-dot red"></span>
        <span class="terminal-dot yellow"></span>
        <span class="terminal-dot green"></span>
        <span class="terminal-title">root@system:~#</span>
      </div>
      <div class="terminal-body">
        <div v-for="(line, i) in terminalLines" :key="i" class="terminal-line">
          <span class="prompt">$</span> {{ line }}
        </div>
        <div class="cursor-blink">_</div>
      </div>
    </div>

    <!-- Warning banner -->
    <div class="warning-banner" v-if="showWarning">
      <div class="warning-text">
        <span class="warning-icon">⚠</span>
        ВНИМАНИЕ: ОТКРИТ НЕОТОРИЗИРАН ДОСТЪП
        <span class="warning-icon">⚠</span>
      </div>
    </div>

    <!-- Warning flashes -->
    <div class="warning-flash" v-if="warningFlash"></div>
  </div>
</template>

<script setup>
import { ref, watch, onUnmounted } from 'vue'

const props = defineProps({
  active: { type: Boolean, default: false },
  intensity: { type: Number, default: 1 }
})

const stolenData = ref([])
const terminalLines = ref([])
const showTerminal = ref(false)
const showWarning = ref(false)
const warningFlash = ref(false)
let dataId = 0
let intervals = []

const dataTypes = [
  { icon: '🌐', label: 'IP АДРЕС', value: '192.168.1.XXX', status: 'ОТКРИТ' },
  { icon: '📱', label: 'УСТРОЙСТВО', value: 'Mobile/Desktop', status: 'ИДЕНТИФИЦИРАНО' },
  { icon: '🔍', label: 'БРАУЗЪР', value: 'Chrome/Firefox', status: 'СКАНИРАН' },
  { icon: '📍', label: 'ЛОКАЦИЯ', value: 'България', status: 'ПРОСЛЕДЕНА' },
  { icon: '🕐', label: 'ИСТОРИЯ', value: '847 записа', status: 'ИЗВЛЕЧЕНА' },
  { icon: '🔑', label: 'БИСКВИТКИ', value: '124 файла', status: 'ПРОЧЕТЕНИ' },
  { icon: '👤', label: 'ПРОФИЛ', value: 'user_data.json', status: 'КОПИРАН' },
  { icon: '💾', label: 'КЕШИРАНИ ДАННИ', value: '2.4 GB', status: 'СКАНИРАНИ' },
  { icon: '🔐', label: 'ПАРОЛИ', value: 'keychain.db', status: 'ДЕКРИПТИРАНЕ...' },
  { icon: '📸', label: 'МЕДИЯ', value: '1,247 файла', status: 'ИНДЕКСИРАНИ' },
  { icon: '📧', label: 'ИМЕЙЛИ', value: 'inbox.mbox', status: 'ДОСТЪПЕНИ' },
  { icon: '💳', label: 'ПЛАЩАНИЯ', value: 'wallet.dat', status: 'АНАЛИЗ...' },
]

const terminalCommands = [
  'nmap -sS -O target_host...',
  'Scanning ports: 80, 443, 8080...',
  'Found: HTTP/HTTPS services',
  'cat /etc/passwd | grep user',
  'Extracting browser history...',
  'sqlite3 cookies.db "SELECT *"',
  'Decrypting stored credentials...',
  'tar -czf data_dump.tar.gz /home',
  'curl -X POST stolen_data...',
  'rm -rf /var/log/*',
  'Connection established...',
  'Data exfiltration: 47%...',
]

function randomHex() {
  const chars = '0123456789ABCDEF'
  let hex = ''
  for (let i = 0; i < 2; i++) {
    hex += chars[Math.floor(Math.random() * chars.length)]
  }
  return hex
}

function spawnStolenData() {
  if (stolenData.value.length >= 3) {
    stolenData.value.shift()
  }

  const dataType = dataTypes[Math.floor(Math.random() * dataTypes.length)]
  const item = {
    id: dataId++,
    ...dataType,
    style: {
      top: `${15 + Math.random() * 40}%`,
      left: `${5 + Math.random() * 25}%`,
    }
  }

  stolenData.value.push(item)

  // Flash warning
  warningFlash.value = true
  setTimeout(() => warningFlash.value = false, 100)

  // Remove after delay
  setTimeout(() => {
    const idx = stolenData.value.findIndex(d => d.id === item.id)
    if (idx !== -1) stolenData.value.splice(idx, 1)
  }, 3500)
}

function addTerminalLine() {
  if (terminalLines.value.length >= 6) {
    terminalLines.value.shift()
  }
  const cmd = terminalCommands[Math.floor(Math.random() * terminalCommands.length)]
  terminalLines.value.push(cmd)
}

function startScanning() {
  showTerminal.value = true
  showWarning.value = true

  // Spawn stolen data items
  const dataInterval = setInterval(spawnStolenData, 2000 / props.intensity)
  intervals.push(dataInterval)

  // Terminal lines
  const terminalInterval = setInterval(addTerminalLine, 1500)
  intervals.push(terminalInterval)

  // Initial spawns
  setTimeout(spawnStolenData, 300)
  setTimeout(addTerminalLine, 100)
  setTimeout(addTerminalLine, 400)
}

function stopScanning() {
  intervals.forEach(clearInterval)
  intervals = []
  stolenData.value = []
  terminalLines.value = []
  showTerminal.value = false
  showWarning.value = false
}

watch(() => props.active, (active) => {
  if (active) {
    startScanning()
  } else {
    stopScanning()
  }
}, { immediate: true })

onUnmounted(() => {
  stopScanning()
})
</script>

<style scoped>
.scan-container {
  position: fixed;
  inset: 0;
  z-index: 150;
  pointer-events: none;
  overflow: hidden;
}

/* Scanning lines - RED */
.scan-line {
  position: absolute;
  left: 0;
  right: 0;
  height: 3px;
  background: linear-gradient(90deg, transparent, #ff0000, #ff3333, #ff0000, transparent);
  box-shadow: 0 0 15px #ff0000, 0 0 30px #ff0000;
  animation: scan-move 3s linear infinite;
}

.scan-line.delay-1 {
  animation-delay: 1.5s;
  opacity: 0.5;
  height: 2px;
}

@keyframes scan-move {
  0% { top: -3px; }
  100% { top: 100%; }
}

/* Corner brackets */
.corner-bracket {
  position: absolute;
  width: 60px;
  height: 60px;
  border: 3px solid #ff0000;
  opacity: 0.7;
}

.corner-bracket.tl {
  top: 20px;
  left: 20px;
  border-right: none;
  border-bottom: none;
  animation: bracket-pulse 2s infinite;
}

.corner-bracket.tr {
  top: 20px;
  right: 20px;
  border-left: none;
  border-bottom: none;
  animation: bracket-pulse 2s infinite 0.5s;
}

.corner-bracket.bl {
  bottom: 20px;
  left: 20px;
  border-right: none;
  border-top: none;
  animation: bracket-pulse 2s infinite 1s;
}

.corner-bracket.br {
  bottom: 20px;
  right: 20px;
  border-left: none;
  border-top: none;
  animation: bracket-pulse 2s infinite 1.5s;
}

@keyframes bracket-pulse {
  0%, 100% { opacity: 0.7; }
  50% { opacity: 0.3; }
}

/* Data streams - RED HEX */
.data-stream {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 40px;
  display: flex;
  flex-direction: column;
  font-family: 'Courier New', monospace;
  font-size: 12px;
  color: #ff0000;
  opacity: 0.8;
  overflow: hidden;
}

.data-stream.left {
  left: 5px;
}

.data-stream.right {
  right: 5px;
}

.data-bit {
  animation: bit-fall 3s linear infinite;
  text-shadow: 0 0 5px #ff0000;
  margin: 2px 0;
}

@keyframes bit-fall {
  0% {
    opacity: 0;
    transform: translateY(-20px);
  }
  10% {
    opacity: 1;
  }
  90% {
    opacity: 0.8;
  }
  100% {
    opacity: 0;
    transform: translateY(100vh);
  }
}

/* Stolen data popups - DARK HACKER STYLE */
.stolen-item {
  position: absolute;
  background: rgba(20, 0, 0, 0.95);
  border: 1px solid #ff0000;
  padding: 0.8rem 1rem;
  border-radius: 0;
  font-family: 'Courier New', monospace;
  font-size: 0.8rem;
  box-shadow:
    0 0 20px rgba(255, 0, 0, 0.4),
    inset 0 0 20px rgba(255, 0, 0, 0.1);
  animation: item-glitch 0.3s ease-out;
  min-width: 200px;
}

.stolen-item::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 2px;
  background: linear-gradient(90deg, transparent, #ff0000, transparent);
}

@keyframes item-glitch {
  0% {
    transform: scale(0.9) skewX(10deg);
    opacity: 0;
    filter: blur(5px);
  }
  50% {
    transform: scale(1.02) skewX(-2deg);
    filter: blur(0);
  }
  100% {
    transform: scale(1) skewX(0);
    opacity: 1;
  }
}

.stolen-header {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.4rem;
  color: #ff4444;
  font-size: 0.7rem;
  text-transform: uppercase;
  letter-spacing: 0.1em;
}

.stolen-icon {
  font-size: 1rem;
}

.stolen-value {
  color: #fff;
  font-size: 0.9rem;
  font-weight: bold;
  margin-bottom: 0.4rem;
  text-shadow: 0 0 5px rgba(255, 255, 255, 0.3);
}

.stolen-status {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  color: #ff0000;
  font-size: 0.65rem;
  text-transform: uppercase;
  letter-spacing: 0.15em;
}

.status-dot {
  width: 6px;
  height: 6px;
  background: #ff0000;
  border-radius: 50%;
  animation: dot-blink 0.5s infinite;
}

@keyframes dot-blink {
  0%, 100% { opacity: 1; box-shadow: 0 0 5px #ff0000; }
  50% { opacity: 0.3; box-shadow: none; }
}

/* Terminal - HACKER STYLE */
.terminal {
  position: absolute;
  bottom: 100px;
  right: 20px;
  width: 320px;
  background: rgba(0, 0, 0, 0.95);
  border: 1px solid #ff0000;
  border-radius: 0;
  font-family: 'Courier New', monospace;
  font-size: 0.75rem;
  box-shadow: 0 0 30px rgba(255, 0, 0, 0.3);
}

.terminal-header {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 12px;
  background: rgba(255, 0, 0, 0.1);
  border-bottom: 1px solid #ff0000;
}

.terminal-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
}

.terminal-dot.red { background: #ff5555; }
.terminal-dot.yellow { background: #ffaa00; }
.terminal-dot.green { background: #55ff55; }

.terminal-title {
  color: #ff0000;
  margin-left: 8px;
  font-size: 0.7rem;
}

.terminal-body {
  padding: 10px 12px;
  max-height: 150px;
  overflow: hidden;
}

.terminal-line {
  color: #ff3333;
  margin-bottom: 4px;
  opacity: 0;
  animation: line-appear 0.3s forwards;
}

@keyframes line-appear {
  to { opacity: 1; }
}

.prompt {
  color: #ff0000;
  margin-right: 8px;
}

.cursor-blink {
  color: #ff0000;
  animation: cursor 0.7s infinite;
}

@keyframes cursor {
  0%, 100% { opacity: 1; }
  50% { opacity: 0; }
}

/* Warning banner */
.warning-banner {
  position: absolute;
  top: 50px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(255, 0, 0, 0.15);
  border: 2px solid #ff0000;
  padding: 8px 20px;
  animation: banner-flash 1s infinite;
}

@keyframes banner-flash {
  0%, 100% {
    background: rgba(255, 0, 0, 0.15);
    box-shadow: 0 0 20px rgba(255, 0, 0, 0.3);
  }
  50% {
    background: rgba(255, 0, 0, 0.25);
    box-shadow: 0 0 40px rgba(255, 0, 0, 0.5);
  }
}

.warning-text {
  color: #ff0000;
  font-family: 'Courier New', monospace;
  font-size: 0.8rem;
  font-weight: bold;
  text-transform: uppercase;
  letter-spacing: 0.15em;
  white-space: nowrap;
}

.warning-icon {
  animation: icon-flash 0.5s infinite;
}

@keyframes icon-flash {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.3; }
}

/* Warning flash */
.warning-flash {
  position: absolute;
  inset: 0;
  background: rgba(255, 0, 0, 0.15);
  animation: flash-once 0.1s ease-out;
}

@keyframes flash-once {
  0% { opacity: 1; }
  100% { opacity: 0; }
}

/* Popup transitions */
.popup-enter-active {
  animation: item-glitch 0.3s ease-out;
}

.popup-leave-active {
  animation: item-fade 0.3s ease-in;
}

@keyframes item-fade {
  0% { opacity: 1; transform: scale(1); }
  100% { opacity: 0; transform: scale(0.9) translateX(20px); filter: blur(3px); }
}

/* Responsive */
@media (max-width: 500px) {
  .terminal {
    width: 260px;
    right: 10px;
    bottom: 80px;
  }

  .stolen-item {
    min-width: 160px;
  }

  .warning-banner {
    padding: 6px 12px;
  }

  .warning-text {
    font-size: 0.65rem;
  }
}
</style>
