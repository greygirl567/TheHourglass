<script setup>
import { ref, watch, computed, onBeforeUnmount } from 'vue'

const props = defineProps({
  progress: Number,
  isRunning: Boolean,
  statusText: String,
  showCompletedMessage: Boolean
})

const statusDotClass = computed(() => {
  if (props.statusText === 'Ready') return 'status-dot'
  if (props.statusText === 'In Motion') return 'status-dot active'
  return 'status-dot completed'
})

const particleIntervalId = ref(null)

// 创建沙粒粒子效果
function createSandParticles() {
  if (particleIntervalId.value) clearInterval(particleIntervalId.value)

  particleIntervalId.value = setInterval(() => {
    if (!props.isRunning) return

    const sandParticles = document.getElementById('sandParticles')
    if (!sandParticles) return

    for (let i = 0; i < 4; i++) {
      const particle = document.createElement('div')
      particle.classList.add('particle')

      // 随机位置和大小
      const left = Math.random() * 25 - 12.5
      const size = 1 + Math.random() * 4
      const duration = 0.5 + Math.random() * 0.4

      particle.style.left = `calc(50% + ${left}px)`
      particle.style.top = '45%'
      particle.style.width = `${size}px`
      particle.style.height = `${size}px`
      particle.style.animation = `falling-particle ${duration}s linear forwards`

      sandParticles.appendChild(particle)

      // 移除粒子
      setTimeout(() => {
        if (sandParticles.contains(particle)) {
          sandParticles.removeChild(particle)
        }
      }, duration * 1000)
    }
  }, 150)
}

// 响应沙漏状态变化
watch(() => props.isRunning, (newVal) => {
  const sandFlow = document.getElementById('sandFlow')
  if (sandFlow) {
    sandFlow.style.display = newVal ? 'block' : 'none'
  }

  if (newVal) {
    createSandParticles()
  } else {
    clearInterval(particleIntervalId.value)
    particleIntervalId.value = null

    const sandParticles = document.getElementById('sandParticles')
    if (sandParticles) sandParticles.innerHTML = ''
  }
})

// 清理资源
onBeforeUnmount(() => {
  clearInterval(particleIntervalId.value)
})
</script>

<template>
  <div class="sandglass-container">
    <div class="sandglass">
      <div class="glass-container">
        <div class="upper-glass">
          <div
            class="sand upper-sand"
            id="upperSand"
            :style="{ height: `${100 - progress}%` }"
          ></div>
        </div>
        <div class="neck"></div>
        <div class="sand-flow" id="sandFlow"></div>
        <div class="sand-particles" id="sandParticles"></div>
        <div class="lower-glass">
          <div
            class="sand lower-sand"
            id="lowerSand"
            :style="{ height: `${progress}%` }"
          ></div>
          <div
            class="sand-pile"
            id="sandPile"
            :style="{ height: `${Math.min(30, progress * 0.33)}px` }"
          ></div>
        </div>
      </div>
    </div>

    <div class="status-indicator">
      <div :class="statusDotClass"></div>
      <span class="status-text">{{ statusText }}</span>
    </div>
    <div
      v-if="showCompletedMessage"
      class="completed-message"
    >
      TIME OUT!
    </div>
  </div>
</template>
