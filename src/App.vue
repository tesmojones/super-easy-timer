<template>
  <v-app>
    <v-container class="pa-0" fluid>
      <v-sheet :height="isMiniMode ? 80 : 200" :width="isMiniMode ? 200 : 300" class="d-flex flex-column align-center justify-center rounded-lg position-relative overflow-hidden" color="rgba(25, 25, 25, 0.95)" style="-webkit-app-region: drag">
        <!-- Window Controls -->
        <div class="d-flex gap-1 position-absolute" style="top: 8px; left: 8px; -webkit-app-region: no-drag">
          <v-btn density="compact" icon size="x-small" color="#ff5f57" variant="flat" @click="window.close()" class="window-control" />
          <v-btn density="compact" icon size="x-small" color="#febc2e" variant="flat" @click="window.minimize()" class="window-control" />
          <v-btn density="compact" icon size="x-small" color="#28c840" variant="flat" @click="window.maximize()" class="window-control" />
        </div>

        <!-- Mini Mode Toggle -->
        <v-btn icon="mdi-fullscreen-exit" size="small" variant="text" class="position-absolute" style="top: 8px; right: 8px; -webkit-app-region: no-drag" color="white" @click="toggleMiniMode" />

        <!-- Timer Controls -->
        <div class="d-flex gap-2" :class="{ 'position-absolute': true }" :style="{ 
          top: isMiniMode ? '8px' : 'auto',
          right: isMiniMode ? '40px' : '50%',
          bottom: isMiniMode ? 'auto' : '0px',
          transform: isMiniMode ? 'none' : 'translateX(50%)',
          '-webkit-app-region': 'no-drag',
          'margin-top': isMiniMode ? '0' : '20px'
        }">
          <v-btn :icon="isRunning ? 'mdi-pause' : 'mdi-play'" :size="isMiniMode ? 'small' : 'large'" color="white" variant="text" @click="isRunning ? pauseTimer() : startTimer()" class="control-btn" />
          <v-btn icon="mdi-stop" :size="isMiniMode ? 'small' : 'large'" color="white" variant="text" @click="resetTimer" class="control-btn" />
        </div>

        <!-- Time Input -->
        <v-text-field v-model="inputValue" placeholder="Enter Time Here (ex. 20 min, add 1 hour, etc)" variant="outlined" density="compact" hide-details class="time-input" bg-color="rgba(0, 0, 0, 0.2)" style="-webkit-app-region: no-drag" @input="handleInputChange" />

        <!-- Timer Display -->
        <div class="timer-display" :class="{ 'mini': isMiniMode }">
          {{ formatTime(time) }}
        </div>
      </v-sheet>
    </v-container>
  </v-app>
</template>

<script setup>
import { ref, onUnmounted, computed } from 'vue'
import * as chrono from 'chrono-node'

const time = ref(0)
const isRunning = ref(false)
const inputValue = ref('')
const intervalId = ref(null)
const isMiniMode = ref(false)

const startTimer = () => {
  if (!isRunning.value && time.value > 0) {
    intervalId.value = setInterval(() => {
      time.value = time.value <= 1 ? 0 : time.value - 1
      if (time.value === 0) {
        clearInterval(intervalId.value)
        isRunning.value = false
      }
    }, 1000)
    isRunning.value = true
  }
}

const pauseTimer = () => {
  if (intervalId.value) {
    clearInterval(intervalId.value)
    intervalId.value = null
  }
  isRunning.value = false
}

const resetTimer = () => {
  if (intervalId.value) {
    clearInterval(intervalId.value)
    intervalId.value = null
  }
  time.value = 0
  isRunning.value = false
  inputValue.value = ''
}

const handleInputChange = () => {
  try {
    const parsed = chrono.parseDate(`in ${inputValue.value}`)
    if (parsed) {
      const seconds = Math.floor((parsed.getTime() - new Date().getTime()) / 1000)
      if (seconds > 0) time.value = seconds
    }
  } catch (error) {
    console.error('Error parsing time:', error)
  }
}

const formatTime = (seconds) => {
  const mins = Math.floor(seconds / 60)
  const secs = seconds % 60
  return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
}

const toggleMiniMode = () => {
  isMiniMode.value = !isMiniMode.value
  window.require('electron').ipcRenderer.send('toggle-mini-mode', isMiniMode.value)
}

onUnmounted(() => {
  if (intervalId.value) clearInterval(intervalId.value)
})
</script>

<style scoped>
.window-control {
  width: 12px !important;
  height: 12px !important;
  min-width: 12px !important;
  border-radius: 50%;
}

.control-btn {
  background-color: rgba(255, 255, 255, 0.05);
}

.control-btn:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.time-input {
  width: 85%;
  position: absolute;
  top: 25px;
}

.time-input :deep(.v-field__input) {
  color: rgba(255, 255, 255, 0.9) !important;
  font-size: 13px;
  padding: 10px 14px;
}

.time-input :deep(.v-field__outline) {
  border-color: rgba(255, 255, 255, 0.1);
}

.time-input:hover :deep(.v-field__outline) {
  border-color: rgba(255, 255, 255, 0.2);
}

.time-input:focus-within :deep(.v-field__outline) {
  border-color: rgba(255, 255, 255, 0.3);
}

.timer-display {
  font-weight: 300;
  color: white;
  font-size: 96px;
  letter-spacing: 4px;
  margin-top: 45px;
  margin-bottom: 0;
  line-height: 1;
}

.timer-display.mini {
  font-size: 64px;
  margin-top: 0;
}
</style>