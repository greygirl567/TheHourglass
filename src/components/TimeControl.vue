<script setup>
import { defineProps, defineEmits } from 'vue';

const props = defineProps({
  hours: Number,
  minutes: Number,
  seconds: Number,
  isRunning: Boolean,
  formattedTime: String,
  progress: Number
});

const emit = defineEmits(['time-update', 'start-timer', 'reset-timer']);

function updateTime(type, value) {
  // 处理空值情况，将空字符串转换为0
  const parsedValue = value === '' ? 0 : parseInt(value) || 0;

  // 使用 props 中的值作为基础
  const newTime = {
    hours: type === 'hours' ? parsedValue : props.hours,
    minutes: type === 'minutes' ? parsedValue : props.minutes,
    seconds: type === 'seconds' ? parsedValue : props.seconds
  };

  // 确保值在合理范围内
  if (type === 'hours') newTime.hours = Math.max(0, Math.min(23, newTime.hours));
  if (type === 'minutes') newTime.minutes = Math.max(0, Math.min(59, newTime.minutes));
  if (type === 'seconds') newTime.seconds = Math.max(0, Math.min(59, newTime.seconds));

  //子组件向父组件传递参数
  emit('time-update', newTime);
}
</script>

<template>
  <div class="time-control">
    <h2 class="control-title">Time Settings</h2>

    <div class="time-inputs">
      <div class="time-input-group">
        <!-- for 属性规定标签绑定到哪个表单元素。 -->
        <label for="hours">Hours</label>
        <input
          type="number"
          id="hours"
          class="time-input"
          min="0"
          max="23"
          :value="hours"
          :disabled="isRunning"
          @input="updateTime('hours', $event.target.value)"
        >
      </div>

      <div class="time-input-group">
        <label for="minutes">Minutes</label>
        <input
          type="number"
          id="minutes"
          class="time-input"
          min="0"
          max="59"
          :value="minutes"
          :disabled="isRunning"
          @input="updateTime('minutes', $event.target.value)"
        >
      </div>

      <div class="time-input-group">
        <label for="seconds">Seconds</label>
        <input
          type="number"
          id="seconds"
          class="time-input"
          min="0"
          max="59"
          :value="seconds"
          :disabled="isRunning"
          @input="updateTime('seconds', $event.target.value)"
        >
      </div>
    </div>

    <div class="timer-display">{{ formattedTime }}</div>

    <div class="progress-container">
      <div class="progress-bar" :style="{ width: `${progress}%` }"></div>
    </div>

    <div class="buttons">
      <button
        id="startBtn"
        class="btn btn-primary"
        @click="$emit('start-timer')"
      >Start Timer</button>
      <button
        id="resetBtn"
        class="btn btn-secondary"
        @click="$emit('reset-timer')"
      >Reset</button>
    </div>
  </div>
</template>
