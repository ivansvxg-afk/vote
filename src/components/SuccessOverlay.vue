<template>
  <Transition name="fade-scale">
    <div v-if="show" class="overlay">
      <div class="success-content">
        <div class="success-icon">✓</div>
        <h2>Благодарим ви!</h2>
        <p>Вашият глас има значение. Вие правите разликата.</p>
        <div class="confetti-container">
          <div v-for="n in 20" :key="'conf-'+n" class="confetti" :style="getConfettiStyle(n)"></div>
        </div>
        <button class="btn btn-yes" @click="$emit('close')">Затвори</button>
      </div>
    </div>
  </Transition>
</template>

<script setup>
defineProps({
  show: { type: Boolean, default: false }
})

defineEmits(['close'])

function getConfettiStyle(n) {
  return {
    left: `${Math.random() * 100}%`,
    animationDelay: `${Math.random() * 2}s`,
    backgroundColor: ['#00c8b3', '#00968a', '#fff', '#ffd700'][n % 4]
  }
}
</script>

<style scoped>
.overlay {
  position: fixed;
  inset: 0;
  display: grid;
  place-items: center;
  background: rgba(0, 0, 0, 0.9);
  z-index: 1001;
}

.success-content {
  background: linear-gradient(135deg, #f0fff0, #e8f5e9);
  color: #111;
  padding: 2.5rem;
  border-radius: 16px;
  width: min(500px, calc(100% - 2rem));
  text-align: center;
  position: relative;
  overflow: hidden;
}

.success-icon {
  width: 80px;
  height: 80px;
  background: linear-gradient(135deg, #00c8b3, #00968a);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 3rem;
  color: white;
  margin: 0 auto 1.5rem;
  animation: success-pop 0.5s ease-out;
}

@keyframes success-pop {
  0% { transform: scale(0); }
  50% { transform: scale(1.2); }
  100% { transform: scale(1); }
}

h2 {
  margin-bottom: 1rem;
  color: #00968a;
  font-size: 1.8rem;
}

p {
  margin-bottom: 1.5rem;
  font-size: 1.1rem;
  color: #333;
}

.btn {
  padding: 0.8rem 2rem;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  cursor: pointer;
}

.btn-yes {
  background: linear-gradient(135deg, #00c8b3, #00968a);
  color: white;
}

.confetti-container {
  position: absolute;
  inset: 0;
  pointer-events: none;
  overflow: hidden;
}

.confetti {
  position: absolute;
  top: -10px;
  width: 10px;
  height: 10px;
  border-radius: 2px;
  animation: confetti-fall 3s ease-out infinite;
}

@keyframes confetti-fall {
  0% { transform: translateY(0) rotate(0deg); opacity: 1; }
  100% { transform: translateY(400px) rotate(720deg); opacity: 0; }
}

.fade-scale-enter-active { animation: fade-in 0.4s ease-out; }
.fade-scale-leave-active { animation: fade-out 0.3s ease-in; }

@keyframes fade-in {
  0% { opacity: 0; transform: scale(0.9); }
  100% { opacity: 1; transform: scale(1); }
}

@keyframes fade-out {
  0% { opacity: 1; transform: scale(1); }
  100% { opacity: 0; transform: scale(0.9); }
}
</style>
