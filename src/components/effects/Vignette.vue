<template>
  <div class="vignette" :style="vignetteStyle"></div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  intensity: { type: Number, default: 0 }, // 0-1
  color: { type: String, default: '#000000' }
})

const vignetteStyle = computed(() => ({
  background: `radial-gradient(ellipse at center,
    transparent 0%,
    transparent ${60 - props.intensity * 40}%,
    ${props.color}${Math.round(props.intensity * 0.7 * 255).toString(16).padStart(2, '0')} ${80 - props.intensity * 20}%,
    ${props.color} 100%)`
}))
</script>

<style scoped>
.vignette {
  position: fixed;
  inset: 0;
  z-index: 100;
  pointer-events: none;
  transition: background 0.5s ease;
}
</style>
