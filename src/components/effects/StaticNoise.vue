<template>
  <div v-if="intensity > 0" class="static-overlay" :style="{ opacity: intensity }"></div>
</template>

<script setup>
defineProps({
  intensity: { type: Number, default: 0 } // 0-1
})
</script>

<style scoped>
.static-overlay {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 100;
  background: repeating-radial-gradient(#000 0 0.0001%, #fff 0 0.0002%) 50% 0/2500px 2500px,
              repeating-conic-gradient(#000 0 0.0001%, #fff 0 0.0002%) 60% 60%/2500px 2500px;
  background-blend-mode: difference;
  animation: static-noise 0.2s steps(8) infinite;
}

@keyframes static-noise {
  to { background-position: 50% 50%, 60% 50%; }
}
</style>
