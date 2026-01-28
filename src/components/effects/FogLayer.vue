<template>
  <div v-if="active" class="fog-container">
    <div class="fog fog-1"></div>
    <div class="fog fog-2"></div>
    <div class="fog fog-3"></div>
  </div>
</template>

<script setup>
defineProps({
  active: { type: Boolean, default: false },
  intensity: { type: Number, default: 0.5 }
})
</script>

<style scoped>
.fog-container {
  position: fixed;
  inset: 0;
  z-index: 80;
  pointer-events: none;
  overflow: hidden;
}

.fog {
  position: absolute;
  width: 200%;
  height: 200%;
  background: radial-gradient(ellipse at center,
    rgba(80, 0, 0, 0.4) 0%,
    rgba(40, 0, 0, 0.2) 30%,
    transparent 70%
  );
  animation: fog-drift 20s ease-in-out infinite;
}

.fog-1 {
  top: -50%;
  left: -50%;
  animation-delay: 0s;
  opacity: 0.6;
}

.fog-2 {
  top: -30%;
  left: -30%;
  animation-delay: -7s;
  animation-duration: 25s;
  opacity: 0.4;
  background: radial-gradient(ellipse at center,
    rgba(100, 0, 0, 0.3) 0%,
    rgba(50, 0, 0, 0.15) 40%,
    transparent 70%
  );
}

.fog-3 {
  top: -40%;
  left: -60%;
  animation-delay: -14s;
  animation-duration: 30s;
  opacity: 0.5;
  background: radial-gradient(ellipse at center,
    rgba(60, 0, 0, 0.35) 0%,
    transparent 60%
  );
}

@keyframes fog-drift {
  0%, 100% {
    transform: translate(0, 0) rotate(0deg) scale(1);
  }
  25% {
    transform: translate(10%, 5%) rotate(2deg) scale(1.1);
  }
  50% {
    transform: translate(-5%, 10%) rotate(-1deg) scale(0.95);
  }
  75% {
    transform: translate(-10%, -5%) rotate(1deg) scale(1.05);
  }
}
</style>
