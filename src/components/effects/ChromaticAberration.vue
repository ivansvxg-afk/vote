<template>
  <div v-if="intensity > 0" class="chromatic-container">
    <div class="chromatic-layer red" :style="redStyle"></div>
    <div class="chromatic-layer cyan" :style="cyanStyle"></div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  intensity: { type: Number, default: 0 } // 0-1
})

const offset = computed(() => props.intensity * 8)

const redStyle = computed(() => ({
  transform: `translateX(${-offset.value}px)`,
  opacity: props.intensity * 0.5
}))

const cyanStyle = computed(() => ({
  transform: `translateX(${offset.value}px)`,
  opacity: props.intensity * 0.5
}))
</script>

<style scoped>
.chromatic-container {
  position: fixed;
  inset: 0;
  z-index: 95;
  pointer-events: none;
  mix-blend-mode: screen;
}

.chromatic-layer {
  position: absolute;
  inset: 0;
  transition: transform 0.1s ease, opacity 0.3s ease;
}

.chromatic-layer.red {
  background: rgba(255, 0, 0, 0.1);
  mix-blend-mode: multiply;
}

.chromatic-layer.cyan {
  background: rgba(0, 255, 255, 0.1);
  mix-blend-mode: multiply;
}
</style>
