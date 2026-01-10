<template>
  <div ref="container" class="nightmare-scene"></div>
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
}

function createEye() {
  const group = new THREE.Group()

  // Outer glow sphere
  const glowGeometry = new THREE.SphereGeometry(2.2, 32, 32)
  const glowMaterial = new THREE.MeshBasicMaterial({
    color: 0xff0000,
    transparent: true,
    opacity: 0.15,
    side: THREE.BackSide
  })
  const glow = new THREE.Mesh(glowGeometry, glowMaterial)
  group.add(glow)

  // Eyeball - creamy white with slight yellow tint
  const eyeGeometry = new THREE.SphereGeometry(1.5, 64, 64)
  const eyeMaterial = new THREE.MeshStandardMaterial({
    color: 0xfffaf0,
    roughness: 0.3,
    metalness: 0.1,
    emissive: 0x331111,
    emissiveIntensity: 0.3
  })
  const eyeball = new THREE.Mesh(eyeGeometry, eyeMaterial)
  group.add(eyeball)

  // Blood veins - more detailed
  for (let i = 0; i < 12; i++) {
    const veinCurve = new THREE.CatmullRomCurve3([
      new THREE.Vector3(0, 0, 1.5),
      new THREE.Vector3(
        (Math.random() - 0.5) * 0.8,
        (Math.random() - 0.5) * 0.8,
        1.3
      ),
      new THREE.Vector3(
        (Math.random() - 0.5) * 1.5,
        (Math.random() - 0.5) * 1.5,
        0.8
      )
    ])
    const veinGeo = new THREE.TubeGeometry(veinCurve, 10, 0.02 + Math.random() * 0.02, 6, false)
    const veinMat = new THREE.MeshBasicMaterial({
      color: new THREE.Color(0.6 + Math.random() * 0.4, 0, 0)
    })
    const vein = new THREE.Mesh(veinGeo, veinMat)
    vein.rotation.z = (Math.PI * 2 / 12) * i
    group.add(vein)
  }

  // Iris - glowing red ring
  const irisGeometry = new THREE.RingGeometry(0.3, 0.8, 64)
  const irisMaterial = new THREE.MeshBasicMaterial({
    color: 0xff2200,
    side: THREE.DoubleSide
  })
  const iris = new THREE.Mesh(irisGeometry, irisMaterial)
  iris.position.z = 1.48
  group.add(iris)

  // Iris glow
  const irisGlowGeo = new THREE.CircleGeometry(0.9, 32)
  const irisGlowMat = new THREE.MeshBasicMaterial({
    color: 0xff0000,
    transparent: true,
    opacity: 0.4
  })
  const irisGlow = new THREE.Mesh(irisGlowGeo, irisGlowMat)
  irisGlow.position.z = 1.46
  group.add(irisGlow)

  // Pupil - vertical slit like a demon/cat
  const pupilShape = new THREE.Shape()
  pupilShape.moveTo(0, -0.35)
  pupilShape.quadraticCurveTo(0.12, 0, 0, 0.35)
  pupilShape.quadraticCurveTo(-0.12, 0, 0, -0.35)

  const pupilGeo = new THREE.ShapeGeometry(pupilShape)
  const pupilMat = new THREE.MeshBasicMaterial({ color: 0x000000 })
  const pupil = new THREE.Mesh(pupilGeo, pupilMat)
  pupil.position.z = 1.5
  group.add(pupil)

  // Inner pupil glow (subtle red core)
  const pupilCore = new THREE.Mesh(
    new THREE.CircleGeometry(0.05, 16),
    new THREE.MeshBasicMaterial({ color: 0xff0000 })
  )
  pupilCore.position.z = 1.51
  group.add(pupilCore)

  // Cornea - transparent bulge
  const corneaGeo = new THREE.SphereGeometry(0.9, 32, 32, 0, Math.PI * 2, 0, Math.PI / 2)
  const corneaMat = new THREE.MeshPhysicalMaterial({
    color: 0xffffff,
    transparent: true,
    opacity: 0.2,
    roughness: 0,
    metalness: 0,
    clearcoat: 1,
    clearcoatRoughness: 0
  })
  const cornea = new THREE.Mesh(corneaGeo, corneaMat)
  cornea.position.z = 1.1
  cornea.rotation.x = -Math.PI / 2
  group.add(cornea)

  // Random position - spread out more
  group.position.set(
    (Math.random() - 0.5) * 50,
    (Math.random() - 0.5) * 35,
    (Math.random() - 0.5) * 30 - 5
  )

  // Random scale
  const scale = 0.8 + Math.random() * 0.6
  group.scale.setScalar(scale)

  // Store movement data
  group.userData = {
    floatSpeed: 0.3 + Math.random() * 0.4,
    floatOffset: Math.random() * Math.PI * 2,
    rotationSpeed: (Math.random() - 0.5) * 0.01,
    pulseSpeed: 1 + Math.random() * 2,
    pulseOffset: Math.random() * Math.PI * 2,
    glow: glow,
    irisGlow: irisGlow,
    pupilCore: pupilCore
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

    // Look at camera (creepy tracking)
    const lookTarget = new THREE.Vector3(
      camera.position.x + Math.sin(time * 1.5) * 3,
      camera.position.y + Math.cos(time * 1.2) * 3,
      camera.position.z
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
</style>
