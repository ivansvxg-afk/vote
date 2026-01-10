<template>
  <div ref="container" class="nightmare-scene"></div>
  <!-- Invisible overlay to capture mouse events over iframe -->
  <div
    v-if="props.eyeCount > 0"
    class="mouse-tracker"
    @mousemove="trackMouse"
    @click="passClick"
  ></div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import * as THREE from 'three'

const props = defineProps({
  intensity: { type: Number, default: 0 }, // 0-1
  eyeCount: { type: Number, default: 0 },
  bloodParticles: { type: Boolean, default: false }
})

const container = ref(null)

let scene, camera, renderer, animationId
let eyes = []
let particles = null
let time = 0
let mouse = { x: 0, y: 0 }

function onMouseMove(event) {
  // Normalize mouse coordinates to -1 to 1
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1
}

// Exposed for template
function trackMouse(event) {
  onMouseMove(event)
}

// Allow clicks to pass through to iframe
function passClick(event) {
  const overlay = event.target
  overlay.style.pointerEvents = 'none'

  const elementBelow = document.elementFromPoint(event.clientX, event.clientY)
  if (elementBelow) {
    elementBelow.click()
  }

  setTimeout(() => {
    overlay.style.pointerEvents = 'auto'
  }, 100)
}

function init() {
  if (!container.value) return

  // Scene
  scene = new THREE.Scene()
  scene.fog = new THREE.FogExp2(0x000000, 0.02)

  // Camera
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.z = 30

  // Renderer
  renderer = new THREE.WebGLRenderer({ alpha: true, antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  container.value.appendChild(renderer.domElement)

  // Ambient light - subtle
  const ambientLight = new THREE.AmbientLight(0x220011, 0.8)
  scene.add(ambientLight)

  // Main red point light
  const pointLight = new THREE.PointLight(0xff2200, 2, 100)
  pointLight.position.set(0, 0, 30)
  scene.add(pointLight)

  // Secondary lights for depth
  const pointLight2 = new THREE.PointLight(0xff0044, 1.5, 80)
  pointLight2.position.set(-20, 10, 15)
  scene.add(pointLight2)

  const pointLight3 = new THREE.PointLight(0x880000, 1, 60)
  pointLight3.position.set(20, -10, 10)
  scene.add(pointLight3)

  // Hemisphere light for subtle fill
  const hemiLight = new THREE.HemisphereLight(0x111111, 0x440000, 0.5)
  scene.add(hemiLight)

  // Create blood particles
  createParticles()

  // Start animation
  animate()

  // Handle resize
  window.addEventListener('resize', onResize)

  // Handle mouse move for eye tracking
  window.addEventListener('mousemove', onMouseMove)
}

function createEye() {
  const group = new THREE.Group()

  // Create canvas texture for realistic eye
  const canvas = document.createElement('canvas')
  canvas.width = 512
  canvas.height = 512
  const ctx = canvas.getContext('2d')

  // Draw eyeball base (gradient white to slight pink)
  const eyeGradient = ctx.createRadialGradient(256, 256, 0, 256, 256, 256)
  eyeGradient.addColorStop(0, '#ffffff')
  eyeGradient.addColorStop(0.7, '#fff5f5')
  eyeGradient.addColorStop(1, '#ffcccc')
  ctx.fillStyle = eyeGradient
  ctx.beginPath()
  ctx.arc(256, 256, 250, 0, Math.PI * 2)
  ctx.fill()

  // Draw blood veins
  ctx.strokeStyle = '#cc0000'
  ctx.lineWidth = 2
  for (let i = 0; i < 15; i++) {
    const angle = (Math.PI * 2 / 15) * i + Math.random() * 0.3
    const startX = 256 + Math.cos(angle) * 80
    const startY = 256 + Math.sin(angle) * 80
    const endX = 256 + Math.cos(angle) * (200 + Math.random() * 50)
    const endY = 256 + Math.sin(angle) * (200 + Math.random() * 50)

    ctx.beginPath()
    ctx.moveTo(startX, startY)
    const cp1x = startX + (Math.random() - 0.5) * 60
    const cp1y = startY + (Math.random() - 0.5) * 60
    ctx.quadraticCurveTo(cp1x, cp1y, endX, endY)
    ctx.stroke()
  }

  // Draw iris (red/orange gradient)
  const irisGradient = ctx.createRadialGradient(256, 256, 0, 256, 256, 80)
  irisGradient.addColorStop(0, '#000000')
  irisGradient.addColorStop(0.3, '#220000')
  irisGradient.addColorStop(0.5, '#660000')
  irisGradient.addColorStop(0.7, '#cc0000')
  irisGradient.addColorStop(0.85, '#ff3300')
  irisGradient.addColorStop(1, '#ff6600')
  ctx.fillStyle = irisGradient
  ctx.beginPath()
  ctx.arc(256, 256, 80, 0, Math.PI * 2)
  ctx.fill()

  // Draw iris pattern (radial lines)
  ctx.strokeStyle = '#440000'
  ctx.lineWidth = 1
  for (let i = 0; i < 36; i++) {
    const angle = (Math.PI * 2 / 36) * i
    ctx.beginPath()
    ctx.moveTo(256 + Math.cos(angle) * 25, 256 + Math.sin(angle) * 25)
    ctx.lineTo(256 + Math.cos(angle) * 75, 256 + Math.sin(angle) * 75)
    ctx.stroke()
  }

  // Draw pupil (vertical slit)
  ctx.fillStyle = '#000000'
  ctx.beginPath()
  ctx.ellipse(256, 256, 8, 35, 0, 0, Math.PI * 2)
  ctx.fill()

  // Add glow to pupil
  const pupilGlow = ctx.createRadialGradient(256, 256, 0, 256, 256, 40)
  pupilGlow.addColorStop(0, 'rgba(255, 0, 0, 0.3)')
  pupilGlow.addColorStop(1, 'rgba(255, 0, 0, 0)')
  ctx.fillStyle = pupilGlow
  ctx.beginPath()
  ctx.arc(256, 256, 40, 0, Math.PI * 2)
  ctx.fill()

  // Specular highlight
  ctx.fillStyle = 'rgba(255, 255, 255, 0.8)'
  ctx.beginPath()
  ctx.ellipse(220, 220, 25, 15, -0.5, 0, Math.PI * 2)
  ctx.fill()

  // Create texture from canvas
  const texture = new THREE.CanvasTexture(canvas)
  texture.needsUpdate = true

  // Eye sphere with texture
  const eyeGeometry = new THREE.SphereGeometry(2, 64, 64)
  const eyeMaterial = new THREE.MeshBasicMaterial({
    map: texture,
  })
  const eyeball = new THREE.Mesh(eyeGeometry, eyeMaterial)
  // Rotate to face forward (fix UV mapping orientation)
  eyeball.rotation.y = Math.PI
  group.add(eyeball)

  // Outer glow
  const glowGeometry = new THREE.SphereGeometry(2.5, 32, 32)
  const glowMaterial = new THREE.MeshBasicMaterial({
    color: 0xff0000,
    transparent: true,
    opacity: 0.15,
    side: THREE.BackSide
  })
  const glow = new THREE.Mesh(glowGeometry, glowMaterial)
  group.add(glow)

  // Random position
  group.position.set(
    (Math.random() - 0.5) * 40,
    (Math.random() - 0.5) * 30,
    (Math.random() - 0.5) * 20 - 10
  )

  // Random scale
  const scale = 1 + Math.random() * 0.8
  group.scale.setScalar(scale)

  // Store movement data
  group.userData = {
    floatSpeed: 0.3 + Math.random() * 0.4,
    floatOffset: Math.random() * Math.PI * 2,
    rotationSpeed: (Math.random() - 0.5) * 0.01,
    pulseSpeed: 1 + Math.random() * 2,
    pulseOffset: Math.random() * Math.PI * 2,
    glow: glow
  }

  return group
}

function createParticles() {
  const particleCount = 800
  const geometry = new THREE.BufferGeometry()
  const positions = new Float32Array(particleCount * 3)
  const colors = new Float32Array(particleCount * 3)
  const sizes = new Float32Array(particleCount)

  for (let i = 0; i < particleCount; i++) {
    positions[i * 3] = (Math.random() - 0.5) * 120
    positions[i * 3 + 1] = (Math.random() - 0.5) * 120
    positions[i * 3 + 2] = (Math.random() - 0.5) * 80

    // Varied red/orange/dark colors for blood/ember effect
    const colorChoice = Math.random()
    if (colorChoice < 0.6) {
      // Blood red
      colors[i * 3] = 0.6 + Math.random() * 0.4
      colors[i * 3 + 1] = 0
      colors[i * 3 + 2] = 0
    } else if (colorChoice < 0.8) {
      // Orange ember
      colors[i * 3] = 1
      colors[i * 3 + 1] = 0.2 + Math.random() * 0.3
      colors[i * 3 + 2] = 0
    } else {
      // Dark ash
      colors[i * 3] = 0.2
      colors[i * 3 + 1] = 0.1
      colors[i * 3 + 2] = 0.1
    }

    sizes[i] = 0.2 + Math.random() * 0.5
  }

  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3))
  geometry.setAttribute('size', new THREE.BufferAttribute(sizes, 1))

  const material = new THREE.PointsMaterial({
    size: 0.4,
    vertexColors: true,
    transparent: true,
    opacity: 0,
    blending: THREE.AdditiveBlending,
    depthWrite: false
  })

  particles = new THREE.Points(geometry, material)
  scene.add(particles)
}

function updateEyes() {
  const targetCount = props.eyeCount

  // Add eyes
  while (eyes.length < targetCount) {
    const eye = createEye()
    eyes.push(eye)
    scene.add(eye)
  }

  // Remove eyes
  while (eyes.length > targetCount) {
    const eye = eyes.pop()
    scene.remove(eye)
  }
}

function animate() {
  animationId = requestAnimationFrame(animate)
  time += 0.016

  // Update eyes
  eyes.forEach(eye => {
    const { floatSpeed, floatOffset, rotationSpeed, pulseSpeed, pulseOffset, glow, irisGlow, pupilCore } = eye.userData

    // Float movement - more organic
    eye.position.y += Math.sin(time * floatSpeed + floatOffset) * 0.025
    eye.position.x += Math.cos(time * floatSpeed * 0.7 + floatOffset) * 0.015
    eye.position.z += Math.sin(time * floatSpeed * 0.5 + floatOffset) * 0.01

    // Slight rotation wobble
    eye.rotation.y += rotationSpeed
    eye.rotation.x = Math.sin(time * 0.5 + floatOffset) * 0.1

    // Pulsing glow effect
    const pulse = 0.5 + Math.sin(time * pulseSpeed + pulseOffset) * 0.5
    if (glow) glow.material.opacity = 0.1 + pulse * 0.15
    if (irisGlow) irisGlow.material.opacity = 0.3 + pulse * 0.3
    if (pupilCore) pupilCore.scale.setScalar(0.8 + pulse * 0.4)

    // Look at mouse cursor (creepy tracking)
    const lookTarget = new THREE.Vector3(
      mouse.x * 30 + eye.position.x * 0.5,  // Offset based on eye position for parallax
      mouse.y * 20 + eye.position.y * 0.5,
      50  // In front of eyes (towards viewer)
    )
    eye.lookAt(lookTarget)
  })

  // Update particles
  if (particles) {
    particles.material.opacity = props.bloodParticles ? props.intensity * 0.6 : 0
    particles.rotation.y += 0.001
    particles.rotation.x += 0.0005

    // Move particles down (falling blood)
    if (props.bloodParticles) {
      const positions = particles.geometry.attributes.position.array
      for (let i = 0; i < positions.length; i += 3) {
        positions[i + 1] -= 0.1 * props.intensity
        if (positions[i + 1] < -50) {
          positions[i + 1] = 50
        }
      }
      particles.geometry.attributes.position.needsUpdate = true
    }
  }

  // Camera shake based on intensity
  if (props.intensity > 0) {
    camera.position.x = Math.sin(time * 10) * props.intensity * 0.5
    camera.position.y = Math.cos(time * 12) * props.intensity * 0.3
  }

  renderer.render(scene, camera)
}

function onResize() {
  if (!camera || !renderer) return
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

function cleanup() {
  if (animationId) cancelAnimationFrame(animationId)
  window.removeEventListener('resize', onResize)
  window.removeEventListener('mousemove', onMouseMove)

  eyes.forEach(eye => scene.remove(eye))
  eyes = []

  if (renderer && container.value) {
    container.value.removeChild(renderer.domElement)
    renderer.dispose()
  }
}

watch(() => props.eyeCount, updateEyes)

onMounted(() => {
  init()
})

onUnmounted(() => {
  cleanup()
})
</script>

<style scoped>
.nightmare-scene {
  position: fixed;
  inset: 0;
  z-index: 50;
  pointer-events: none;
}

.mouse-tracker {
  position: fixed;
  inset: 0;
  z-index: 49;
  pointer-events: auto;
  cursor: default;
}
</style>
