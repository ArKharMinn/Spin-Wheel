<template>
  <div class="wheel-container">
    <div class="wheel-wrapper">
      <canvas ref="wheel" width="500" height="500"></canvas>
      <div class="pointer"></div>
    </div>
    <button @click="spinWheel" :disabled="spinning" class="spin-button">
      {{ spinning ? 'Spinning...' : 'Spin' }}
    </button>
    <p v-if="winner" class="winner-text">
      Winner: <span>{{ winner }}</span>
    </p>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

const wheel = ref<HTMLCanvasElement | null>(null)
const segments = ['Lion 🦁', 'Tiger 🐅', 'Elephant 🐘', 'Giraffe 🦒', 'Monkey 🐒', 'Zebra 🦓']
const spinning = ref(false)
const angle = ref(0)
const winner = ref<string | null>(null)

const drawWheel = () => {
  if (!wheel.value) return
  const ctx = wheel.value.getContext('2d')
  if (!ctx) return

  const size = wheel.value.width
  const center = size / 2
  const radius = center - 10
  const segmentAngle = (2 * Math.PI) / segments.length

  ctx.clearRect(0, 0, size, size)
  ctx.save()
  ctx.shadowColor = 'rgba(0,0,0,0.3)'
  ctx.shadowBlur = 10

  segments.forEach((segment, i) => {
    const startAngle = i * segmentAngle + angle.value - Math.PI / 2
    const endAngle = startAngle + segmentAngle

    const gradient = ctx.createLinearGradient(0, 0, size, size)
    gradient.addColorStop(0, i % 2 === 0 ? '#FFCC00' : '#FF6600')
    gradient.addColorStop(1, i % 2 === 0 ? '#FFD700' : '#FF4500')
    ctx.fillStyle = gradient

    ctx.beginPath()
    ctx.moveTo(center, center)
    ctx.arc(center, center, radius, startAngle, endAngle)
    ctx.closePath()
    ctx.fill()
    ctx.strokeStyle = '#333'
    ctx.lineWidth = 2
    ctx.stroke()

    ctx.save()
    ctx.translate(center, center)
    ctx.rotate(startAngle + segmentAngle / 2)
    ctx.textAlign = 'right'
    ctx.textBaseline = 'middle'
    ctx.fillStyle = '#000'
    ctx.font = 'bold 20px Arial'
    ctx.fillText(segment, radius - 20, 0)
    ctx.restore()
  })

  ctx.beginPath()
  ctx.arc(center, center, 40, 0, 2 * Math.PI)
  ctx.fillStyle = '#fff'
  ctx.strokeStyle = '#333'
  ctx.lineWidth = 3
  ctx.fill()
  ctx.stroke()

  ctx.restore()
}

const spinWheel = () => {
  if (spinning.value) return
  spinning.value = true
  winner.value = null

  const spins = Math.random() * 10 + 10
  const duration = 3000
  const start = performance.now()

  const animate = (time: number) => {
    const elapsed = time - start
    const progress = Math.min(elapsed / duration, 1)
    const eased = 1 - Math.pow(1 - progress, 3)
    angle.value = spins * 2 * Math.PI * eased
    drawWheel()

    if (progress < 1) {
      requestAnimationFrame(animate)
    } else {
      spinning.value = false
      calculateWinner()
    }
  }

  requestAnimationFrame(animate)
}

const calculateWinner = () => {
  const segmentAngle = (2 * Math.PI) / segments.length
  const normalizedAngle = angle.value % (2 * Math.PI)
  const index = Math.floor((segments.length - normalizedAngle / segmentAngle) % segments.length)
  winner.value = segments[index]
}

onMounted(() => {
  drawWheel()
})
</script>

<style scoped>
.wheel-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  font-family: 'Arial', sans-serif;
}

.wheel-wrapper {
  position: relative;
  width: 500px;
  height: 500px;
}

canvas {
  border-radius: 50%;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
}

.pointer {
  position: absolute;
  top: -15px;
  left: 50%;
  transform: translateX(-50%);
  width: 0;
  height: 0;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent;
  border-top: 40px solid indigo;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.5);
}

.spin-button {
  padding: 12px 30px;
  font-size: 18px;
  background: linear-gradient(45deg, #ff6600, #ffcc00);
  border: none;
  border-radius: 12px;
  color: #fff;
  cursor: pointer;
  font-weight: bold;
  transition:
    transform 0.2s,
    box-shadow 0.2s;
}
.spin-button:hover {
  transform: scale(1.05);
  box-shadow: 0 6px 15px rgba(0, 0, 0, 0.3);
}
.spin-button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.winner-text {
  font-size: 22px;
  font-weight: bold;
  color: #ff4500;
  margin-top: 10px;
}
.winner-text span {
  font-size: 26px;
  color: #008000;
}
</style>
