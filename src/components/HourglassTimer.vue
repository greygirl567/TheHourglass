<script setup>
import { ref, computed, onBeforeUnmount } from 'vue'
import TimeControl from './TimeControl.vue'
import Sandglass from './TheSandglass.vue'

// 状态变量
const hours = ref(0)
const minutes = ref(1)
const seconds = ref(0)
const isRunning = ref(false)
const statusText = ref('Ready')
const showCompletedMessage = ref(false)
const progress = ref(0)
const originalTime = ref(60)
const startTime = ref(0)
const timerId = ref(null)
const animationFrameId = ref(null)
const particleIntervalId = ref(null)

// 保存初始时间的变量
const initialHours = ref(0)
const initialMinutes = ref(1)
const initialSeconds = ref(0)

// 计算总秒数
const totalSeconds = computed(() => {
  return hours.value * 3600 + minutes.value * 60 + seconds.value
})

// 格式化时间显示
const formattedTime = computed(() => {
  const h = Math.floor(totalSeconds.value / 3600)
  const m = Math.floor((totalSeconds.value % 3600) / 60)
  const s = totalSeconds.value % 60
  return `${h.toString().padStart(2, '0')}:${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}`
})

// 处理时间输入变化
function handleTimeInput(newTime) {
  hours.value = newTime.hours
  minutes.value = newTime.minutes
  seconds.value = newTime.seconds

  // 更新初始时间
  initialHours.value = newTime.hours
  initialMinutes.value = newTime.minutes
  initialSeconds.value = newTime.seconds

  if (!isRunning.value) {
    originalTime.value = totalSeconds.value
  }
}

// 启动计时器
function startTimer() {
  if (isRunning.value || totalSeconds.value <= 0) return

  isRunning.value = true
  statusText.value = 'In Motion'
  showCompletedMessage.value = false
  startTime.value = Date.now()
  originalTime.value = totalSeconds.value

  // 保存初始时间值
  initialHours.value = hours.value
  initialMinutes.value = minutes.value
  initialSeconds.value = seconds.value

  // 开始倒计时
  timerId.value = setInterval(() => {
    if (totalSeconds.value > 0) {
      const newTotalSeconds = totalSeconds.value - 1

      // 更新实时时间（只在计时显示区域）
      hours.value = Math.floor(newTotalSeconds / 3600)
      minutes.value = Math.floor((newTotalSeconds % 3600) / 60)
      seconds.value = newTotalSeconds % 60

      if (newTotalSeconds === 0) {
        clearInterval(timerId.value)
        isRunning.value = false
        statusText.value = 'Completed'
        showCompletedMessage.value = true
        progress.value = 100

        // 停止粒子效果
        if (particleIntervalId.value) clearInterval(particleIntervalId.value)
      }
    }
  }, 1000)

  // 启动沙粒流动动画
  animateSand()

  // 创建沙粒粒子效果
  createSandParticles()
}

// 沙漏动画
function animateSand() {
  if (!isRunning.value) return

  const elapsed = Date.now() - startTime.value
  // 确保进度百分比不会超过100%，除法计算当前经过的时间占整个计时周期的比例
  progress.value = Math.min(100, (elapsed / (originalTime.value * 1000)) * 100)

  if (progress.value < 100) {
    animationFrameId.value = requestAnimationFrame(animateSand)
  }
}

// 重置计时器
function resetTimer() {
  clearInterval(timerId.value)
  cancelAnimationFrame(animationFrameId.value)
  clearInterval(particleIntervalId.value)

  isRunning.value = false
  statusText.value = 'Ready'
  showCompletedMessage.value = false
  progress.value = 0

  // 重置时间
  hours.value = initialHours.value
  minutes.value = initialMinutes.value
  seconds.value = initialSeconds.value
  originalTime.value = initialHours.value * 3600 + initialMinutes.value * 60 + initialSeconds.value
}

// 创建沙粒粒子效果
function createSandParticles() {
  if (particleIntervalId.value) clearInterval(particleIntervalId.value)

  particleIntervalId.value = setInterval(() => {
    if (!isRunning.value) return

    const sandParticles = document.getElementById('sandParticles')
    if (!sandParticles) return

    for (let i = 0; i < 4; i++) {
      const particle = document.createElement('div')
      particle.classList.add('particle')

      // 随机位置
      const left = Math.random() * 25 - 12.5
      particle.style.left = `calc(50% + ${left}px)`
      particle.style.top = '45%'

      // 随机大小
      const size = 1 + Math.random() * 4
      particle.style.width = `${size}px`
      particle.style.height = `${size}px`

      // 随机下落时间
      const duration = 0.5 + Math.random() * 0.4
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

// 清理资源
onBeforeUnmount(() => {
  clearInterval(timerId.value)
  cancelAnimationFrame(animationFrameId.value)
  clearInterval(particleIntervalId.value)
})
</script>

<template>
  <div class="container">
    <header>
      <h1>Hourglass Timer</h1>
      <p class="subtitle">"Man vergibt keinem Künstler, dass man ihn mit einem Tage verliert, der nicht getanzt hat."</p>
    </header>

    <div class="content">
      <TimeControl
        :hours="initialHours"
        :minutes="initialMinutes"
        :seconds="initialSeconds"
        :isRunning="isRunning"
        :formattedTime="formattedTime"
        :progress="progress"
        @time-update="handleTimeInput"
        @start-timer="startTimer"
        @reset-timer="resetTimer"
      />

      <Sandglass
        :progress="progress"
        :isRunning="isRunning"
        :statusText="statusText"
        :showCompletedMessage="showCompletedMessage"
      />
    </div>

    <footer>
      <p>© 2025 Hourglass</p>
    </footer>
  </div>
</template>
