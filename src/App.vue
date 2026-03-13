<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const countdown = ref({
  hours: 0,
  minutes: 0,
  seconds: 0,
  milliseconds: 0
})

const targetTime = ref(null)
const timerInterval = ref(null)
const pulsePosition = ref(0)
const pulseAnimation = ref(null)

// 雪花效果
const snowflakes = ref([])

const createSnowflakes = () => {
  const snowflakeArray = []
  const snowflakeChars = ['❄', '❅', '❆', '✻', '✼', '❉']
  for (let i = 0; i < 80; i++) {
    snowflakeArray.push({
      id: i,
      x: Math.random() * 100,
      char: snowflakeChars[Math.floor(Math.random() * snowflakeChars.length)],
      size: Math.random() * 20 + 10,
      duration: Math.random() * 5 + 5,
      delay: Math.random() * 10,
      opacity: Math.random() * 0.6 + 0.4,
      drift: Math.random() * 100 - 50
    })
  }
  snowflakes.value = snowflakeArray
}

// 初始化倒计时 - 目标时间：2026 年 3 月 15 日 12:27
const initCountdown = () => {
  // 使用本地时间设置，避免时区问题
  targetTime.value = new Date(2026, 2, 15, 12, 27, 0) // 月份从 0 开始，2 代表 3 月
  
  if (timerInterval.value) clearInterval(timerInterval.value)
  timerInterval.value = setInterval(updateCountdown, 10)
  
  // 启动心电图脉冲动画
  startPulseAnimation()
}

const updateCountdown = () => {
  if (!targetTime.value) return
  
  const now = new Date()
  const diff = targetTime.value.getTime() - now.getTime()
  
  if (diff <= 0) {
    countdown.value = { hours: 0, minutes: 0, seconds: 0, milliseconds: 0 }
    if (timerInterval.value) clearInterval(timerInterval.value)
    return
  }
  
  countdown.value.hours = Math.floor(diff / 3600000)
  countdown.value.minutes = Math.floor((diff % 3600000) / 60000)
  countdown.value.seconds = Math.floor((diff % 60000) / 1000)
  countdown.value.milliseconds = diff % 1000
}

const startPulseAnimation = () => {
  if (pulseAnimation.value) cancelAnimationFrame(pulseAnimation.value)
  
  const animate = () => {
    const now = Date.now()
    // 1 秒周期
    pulsePosition.value = (now % 1000) / 1000
    pulseAnimation.value = requestAnimationFrame(animate)
  }
  animate()
}

const formatNumber = (num, digits = 2) => {
  return num.toString().padStart(digits, '0')
}

// 倒计时到 2026 年 3 月 15 日 12:27
onMounted(() => {
  createSnowflakes()
  initCountdown()
})

onUnmounted(() => {
  if (timerInterval.value) clearInterval(timerInterval.value)
  if (pulseAnimation.value) cancelAnimationFrame(pulseAnimation.value)
})
</script>

<template>
  <div class="container">
    <!-- 雪花背景 -->
    <div class="particles">
      <div 
        v-for="snowflake in snowflakes" 
        :key="snowflake.id"
        class="snowflake"
        :style="{
          left: snowflake.x + '%',
          fontSize: snowflake.size + 'px',
          opacity: snowflake.opacity,
          animationDuration: snowflake.duration + 's',
          animationDelay: snowflake.delay + 's',
          '--drift': snowflake.drift + 'px'
        }"
      >{{ snowflake.char }}</div>
    </div>
    
    <!-- 心电图区域 -->
    <div class="ecg-container">
      <!-- 左边的心 -->
      <div class="heart heart-left">
        <svg viewBox="0 0 24 24" fill="currentColor">
          <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/>
        </svg>
      </div>
      
      <!-- 心电图线条 -->
      <div class="ecg-line">
        <div class="ecg-track">
          <div class="ecg-pulse" :style="{ left: pulsePosition * 100 + '%' }"></div>
        </div>
        <svg class="ecg-wave" viewBox="0 0 500 100" preserveAspectRatio="none">
          <defs>
            <linearGradient id="ecgGradient" x1="0%" y1="0%" x2="100%" y2="0%">
              <stop offset="0%" style="stop-color:#ff0000;stop-opacity:0" />
              <stop offset="40%" style="stop-color:#ff0000;stop-opacity:1" />
              <stop offset="60%" style="stop-color:#ff0000;stop-opacity:1" />
              <stop offset="100%" style="stop-color:#ff0000;stop-opacity:0" />
            </linearGradient>
            <filter id="glow">
              <feGaussianBlur stdDeviation="3" result="coloredBlur"/>
              <feMerge>
                <feMergeNode in="coloredBlur"/>
                <feMergeNode in="SourceGraphic"/>
              </feMerge>
            </filter>
          </defs>
          <path class="ecg-path" d="M0,50 L50,50 L70,50 L85,20 L95,80 L105,10 L115,90 L125,50 L140,50 L160,50 L175,20 L185,80 L195,10 L205,90 L215,50 L230,50 L250,50 L265,20 L275,80 L285,10 L295,90 L305,50 L320,50 L340,50 L355,20 L365,80 L375,10 L385,90 L395,50 L410,50 L430,50 L445,20 L455,80 L465,10 L475,90 L485,50 L500,50" />
        </svg>
      </div>
      
      <!-- 右边的心 -->
      <div class="heart heart-right">
        <svg viewBox="0 0 24 24" fill="currentColor">
          <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/>
        </svg>
      </div>
    </div>
    
    <!-- 倒计时区域 -->
    <div class="countdown-container">
      <div class="countdown-title">倒计时</div>
      <div class="countdown">
        <div class="time-segment">
          <span class="number">{{ formatNumber(countdown.hours) }}</span>
          <span class="label">时</span>
        </div>
        <span class="separator">:</span>
        <div class="time-segment">
          <span class="number">{{ formatNumber(countdown.minutes) }}</span>
          <span class="label">分</span>
        </div>
        <span class="separator">:</span>
        <div class="time-segment">
          <span class="number">{{ formatNumber(countdown.seconds) }}</span>
          <span class="label">秒</span>
        </div>
        <span class="separator">:</span>
        <div class="time-segment milliseconds">
          <span class="number">{{ formatNumber(countdown.milliseconds, 3) }}</span>
          <span class="label">毫秒</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  min-height: 100vh;
  width: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: linear-gradient(135deg, #1e0a3c 0%, #4a1942 25%, #2d1b4e 50%, #3a1c5a 75%, #1e0a3c 100%);
  background-size: 400% 400%;
  animation: gradientShift 15s ease infinite;
  padding: 20px;
  position: relative;
  overflow: hidden;
}

@keyframes gradientShift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

/* 雪花效果 */
.particles {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  pointer-events: none;
}

.snowflake {
  position: absolute;
  top: -10px;
  color: #fff;
  text-shadow: 0 0 5px rgba(255, 255, 255, 0.8);
  animation: fall linear infinite;
  opacity: 0.8;
}

@keyframes fall {
  0% {
    transform: translateY(-10px) translateX(0) rotate(0deg);
    opacity: 0;
  }
  10% {
    opacity: 0.8;
  }
  25% {
    transform: translateY(25vh) translateX(var(--drift)) rotate(90deg);
  }
  50% {
    transform: translateY(50vh) translateX(0) rotate(180deg);
  }
  75% {
    transform: translateY(75vh) translateX(calc(var(--drift) * -1)) rotate(270deg);
  }
  90% {
    opacity: 0.8;
  }
  100% {
    transform: translateY(100vh) translateX(0) rotate(360deg);
    opacity: 0;
  }
}

.ecg-container {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 30px;
  margin-bottom: 60px;
  position: relative;
  z-index: 1;
}

.heart {
  width: 100px;
  height: 100px;
  color: #ff0040;
  animation: heartbeat 1s ease-in-out infinite;
  filter: drop-shadow(0 0 25px rgba(255, 0, 64, 0.8)) drop-shadow(0 0 50px rgba(255, 0, 64, 0.4));
}

.heart-left {
  animation-delay: 0s;
}

.heart-right {
  animation-delay: 0.5s;
}

@keyframes heartbeat {
  0%, 100% { 
    transform: scale(1);
  }
  10%, 30% { 
    transform: scale(1.15);
  }
  20%, 40% { 
    transform: scale(1);
  }
}

.ecg-line {
  position: relative;
  width: 400px;
  height: 100px;
}

.ecg-track {
  position: absolute;
  top: 50%;
  left: 0;
  right: 0;
  height: 3px;
  background: linear-gradient(90deg, transparent, rgba(255, 0, 64, 0.3), transparent);
  transform: translateY(-50%);
  border-radius: 3px;
  overflow: visible;
}

.ecg-pulse {
  position: absolute;
  top: 50%;
  width: 40px;
  height: 6px;
  background: linear-gradient(90deg, transparent, #ff0040, #ff4466, #ff0040, transparent);
  transform: translateY(-50%);
  border-radius: 3px;
  box-shadow: 
    0 0 20px #ff0040,
    0 0 40px #ff0040,
    0 0 60px #ff0040;
  opacity: 0.9;
}

.ecg-wave {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.ecg-path {
  fill: none;
  stroke: #ff0040;
  stroke-width: 3;
  stroke-linecap: round;
  stroke-linejoin: round;
  filter: url(#glow);
  opacity: 0.9;
  animation: ecgFlow 1s linear infinite;
}

@keyframes ecgFlow {
  0% {
    stroke-dasharray: 1000;
    stroke-dashoffset: 1000;
  }
  100% {
    stroke-dasharray: 1000;
    stroke-dashoffset: 0;
  }
}

.countdown-container {
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 20px;
  padding: 40px 60px;
  backdrop-filter: blur(10px);
  box-shadow: 
    0 8px 32px rgba(0, 0, 0, 0.5),
    inset 0 1px 0 rgba(255, 255, 255, 0.1);
  position: relative;
  z-index: 1;
}

.countdown-title {
  font-size: 18px;
  color: rgba(255, 255, 255, 0.6);
  letter-spacing: 8px;
  text-transform: uppercase;
  margin-bottom: 25px;
  font-weight: 300;
  text-shadow: 0 0 10px rgba(255, 255, 255, 0.3);
  text-align: center;
}

.countdown {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.time-segment {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: rgba(0, 0, 0, 0.4);
  border-radius: 12px;
  padding: 15px 20px;
  min-width: 80px;
  border: 1px solid rgba(255, 255, 255, 0.08);
  box-shadow: inset 0 2px 10px rgba(0, 0, 0, 0.3);
}

.time-segment.milliseconds {
  min-width: 100px;
}

.number {
  font-size: 48px;
  font-weight: 700;
  color: #00ff88;
  font-family: 'Consolas', 'Monaco', monospace;
  text-shadow: 0 0 20px rgba(0, 255, 136, 0.5);
  letter-spacing: 2px;
}

.milliseconds .number {
  font-size: 32px;
  color: #00d4ff;
  text-shadow: 0 0 20px rgba(0, 212, 255, 0.5);
}

.label {
  font-size: 12px;
  color: rgba(255, 255, 255, 0.5);
  margin-top: 5px;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.separator {
  font-size: 36px;
  color: rgba(255, 255, 255, 0.3);
  font-weight: 300;
}

@media (max-width: 768px) {
  .container {
    padding: 10px;
  }
  
  .ecg-container {
    gap: 15px;
    margin-bottom: 30px;
  }
  
  .heart {
    width: 50px;
    height: 50px;
  }
  
  .ecg-line {
    width: 120px;
    height: 60px;
  }
  
  .countdown-container {
    padding: 20px 25px;
    width: 90%;
    max-width: 350px;
  }
  
  .countdown-title {
    font-size: 14px;
    letter-spacing: 5px;
    margin-bottom: 15px;
  }
  
  .countdown {
    gap: 5px;
  }
  
  .number {
    font-size: 28px;
  }
  
  .milliseconds .number {
    font-size: 20px;
  }
  
  .time-segment {
    padding: 8px 12px;
    min-width: 45px;
  }
  
  .time-segment.milliseconds {
    min-width: 65px;
  }
  
  .label {
    font-size: 10px;
  }
  
  .separator {
    font-size: 20px;
  }
}

@media (max-width: 480px) {
  .ecg-container {
    gap: 8px;
  }
  
  .heart {
    width: 40px;
    height: 40px;
  }
  
  .ecg-line {
    width: 80px;
    height: 50px;
  }
  
  .countdown-container {
    padding: 15px 18px;
  }
  
  .number {
    font-size: 22px;
  }
  
  .milliseconds .number {
    font-size: 16px;
  }
  
  .time-segment {
    padding: 6px 8px;
    min-width: 38px;
  }
  
  .time-segment.milliseconds {
    min-width: 55px;
  }
  
  .label {
    font-size: 9px;
  }
  
  .separator {
    font-size: 16px;
  }
}
</style>
