<template>
  <div v-if="count > 0" class="eyes-container">
    <div
      v-for="n in count"
      :key="'eye-'+n"
      class="floating-eye"
      :style="getEyeStyle(n)"
    >
      <div class="eye-inner">
        <div class="pupil">X</div>
      </div>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  count: { type: Number, default: 0 } // 0-6
})

const positions = [
  { top: '10%', left: '5%' },
  { top: '15%', right: '8%' },
  { top: '40%', left: '3%' },
  { top: '45%', right: '5%' },
  { top: '70%', left: '7%' },
  { top: '75%', right: '4%' },
]

function getEyeStyle(n) {
  const pos = positions[(n - 1) % positions.length]
  return {
    ...pos,
    animationDelay: `${n * 0.5}s`
  }
}
</script>

<style scoped>
.eyes-container {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 60;
}

.floating-eye {
  position: absolute;
  width: 60px;
  height: 40px;
  animation: eye-float 3s ease-in-out infinite;
}

.eye-inner {
  width: 100%;
  height: 100%;
  background: radial-gradient(ellipse, #fff 0%, #ddd 50%, #888 100%);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 0 20px rgba(255, 0, 0, 0.5), inset 0 0 10px rgba(0, 0, 0, 0.3);
  animation: eye-blink 4s ease-in-out infinite;
}

.pupil {
  width: 20px;
  height: 20px;
  border: 1px solid black;
  border-radius: 50%;
  color: black;
  font-weight: 900;
  display: flex;
  align-items: center;
  justify-content: center;
  animation: pupil-move 2s ease-in-out infinite;
}

@keyframes eye-float {
  0%, 100% { transform: translateY(0) rotate(0deg); }
  50% { transform: translateY(-15px) rotate(5deg); }
}

@keyframes eye-blink {
  0%, 45%, 55%, 100% { transform: scaleY(1); }
  50% { transform: scaleY(0.1); }
}

@keyframes pupil-move {
  0%, 100% { transform: translate(0, 0); }
  25% { transform: translate(5px, -3px); }
  75% { transform: translate(-5px, 3px); }
}
</style>
