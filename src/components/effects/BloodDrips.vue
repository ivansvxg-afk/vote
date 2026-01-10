<template>
  <div v-if="active" class="blood-drips">
    <div
      v-for="n in count"
      :key="n"
      class="drip"
      :style="{
        left: (n * (100 / count)) + '%',
        animationDelay: (n * 0.3) + 's',
        animationDuration: speed + 's'
      }"
    ></div>
  </div>
</template>

<script setup>
defineProps({
  active: { type: Boolean, default: false },
  count: { type: Number, default: 12 },
  speed: { type: Number, default: 4 } // seconds
})
</script>

<style scoped>
.blood-drips {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 100%;
  pointer-events: none;
  z-index: 50;
  overflow: hidden;
}

.drip {
  position: absolute;
  top: -50px;
  width: 8px;
  height: 0;
  background: linear-gradient(to bottom, #8b0000, #ff0000, transparent);
  border-radius: 0 0 50% 50%;
  animation: drip 4s ease-in infinite;
}

@keyframes drip {
  0% { height: 0; top: -50px; opacity: 1; }
  70% { height: 150px; opacity: 1; }
  100% { height: 150px; top: 110%; opacity: 0; }
}
</style>
