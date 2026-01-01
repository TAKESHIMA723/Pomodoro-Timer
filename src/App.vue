<script setup>
import { ref, computed } from 'vue'

const WORK_TIME = 25 * 60
const BREAK_TIME = 5 * 60

const timeLeft = ref(WORK_TIME)
const isRunning = ref(false)
const isWorkMode = ref(true)
let timerInterval = null

// 進捗率 (0%〜100%)
const progressPercentage = computed(() => {
  const totalTime = isWorkMode.value ? WORK_TIME : BREAK_TIME
  return (timeLeft.value / totalTime) * 100
})

// 背景色のスタイルを動的に生成
const backgroundStyle = computed(() => {
  return {
    '--progress-height': `${progressPercentage.value}%`,
    '--theme-color': isWorkMode.value ? '#ff6b6b' : '#42b983' // 作業は赤、休憩は緑
  }
})

const formattedTime = computed(() => {
  const minutes = Math.floor(timeLeft.value / 60)
  const seconds = timeLeft.value % 60
  return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
})

const toggleTimer = () => {
  if (isRunning.value) {
    clearInterval(timerInterval)
    isRunning.value = false
  } else {
    isRunning.value = true
    // 1秒ごとに実行
    timerInterval = setInterval(() => {
      if (timeLeft.value > 0) {
        timeLeft.value--
      } else {
        clearInterval(timerInterval)
        isRunning.value = false
        alert(isWorkMode.value ? '休憩の時間です！' : '作業に戻りましょう！')
        switchMode()
      }
    }, 1000)
  }
}

const resetTimer = () => {
  clearInterval(timerInterval)
  isRunning.value = false
  timeLeft.value = isWorkMode.value ? WORK_TIME : BREAK_TIME
}

const switchMode = () => {
  isWorkMode.value = !isWorkMode.value
  resetTimer()
}
</script>

<template>
  <div class="app-container" :style="backgroundStyle">
    
    <div class="background-layer"></div>

    <div class="content">
      <h1 :style="{ color: isWorkMode ? '#fff' : '#2c3e50' }">
        {{ isWorkMode ? 'Work Time' : 'Break Time' }}
      </h1>
      
      <div class="timer-display" :style="{ color: isWorkMode ? '#fff' : '#2c3e50' }">
        {{ formattedTime }}
      </div>

      <div class="controls">
        <button @click="toggleTimer" :class="{ 'btn-work': isWorkMode, 'btn-break': !isWorkMode }">
          {{ isRunning ? '一時停止' : 'スタート' }}
        </button>
        <button @click="resetTimer" class="btn-secondary">リセット</button>
        <button @click="switchMode" class="btn-secondary">
          {{ isWorkMode ? '休憩モードへ' : '作業モードへ' }}
        </button>
      </div>
    </div>
  </div>
</template>

<style>
/* ページ全体の余白をリセット */
html, body, #app {
  height: 100%;
  margin: 0;
  padding: 0;
}
body {
  font-family: 'Helvetica Neue', Arial, sans-serif;
}
</style>

<style scoped>
.app-container {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background-color: #f0f2f5;
}

/*背景アニメーションの核心部分 */
.background-layer {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  /* Vueから渡された進捗率を高さに設定 */
  height: var(--progress-height);
  /* Vueから渡されたテーマカラーを背景色に設定 */
  background-color: var(--theme-color);
  
  /* 重要：高さの変化を1秒かけて直線的に行う＝ヌルっと動く */
  transition: height 1s linear, background-color 0.5s ease;
  
  z-index: 0; /* コンテンツより後ろに配置 */
}

.content {
  position: relative;
  z-index: 1; /* 背景より前に表示 */
  text-align: center;
  padding-top: 10vh; /* 少し下に配置 */
  transition: color 0.5s;
}

.timer-display {
  font-size: 8rem; /* タイマーをより大きく */
  font-weight: bold;
  margin: 20px 0;
  font-variant-numeric: tabular-nums;
  text-shadow: 0 2px 4px rgba(0,0,0,0.1); /* 少し影をつけて見やすく */
}

button {
  font-size: 1.2rem;
  margin: 0 10px;
  padding: 12px 24px;
  cursor: pointer;
  border: none;
  border-radius: 50px;
  font-weight: bold;
  transition: transform 0.1s, background-color 0.3s;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
}
button:active {
  transform: scale(0.98); /* クリック時に少し凹む演出 */
}

/* ボタンのデザイン調整 */
.btn-work {
  background-color: #ff0c0c;
  color: #ffffff;
}
.btn-break {
  background-color: #2afb9d;
  color: #ffffff;
}
.btn-secondary {
  background-color: #ffffff;
  color: #000000;
}
</style>