<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue'
import imageWebp from '/image.webp'
import imageGif from '/unfrozen.gif'

const countdown = ref({
  hours: 0,
  minutes: 0,
  seconds: 0,
  milliseconds: 0
})

const targetTime = ref(null)
const startTime = ref(null)
const timerInterval = ref(null)
const pulsePosition = ref(0)
const pulseAnimation = ref(null)

// 弹窗控制
const showModal = ref(false)
const birthdayInput = ref('')

// 一键解冻
const openModal = () => {
  showModal.value = true
  birthdayInput.value = ''
}

const closeModal = () => {
  showModal.value = false
  birthdayInput.value = ''
}

// 处理解冻
const handleUnfreeze = () => {
  const input = birthdayInput.value.trim()
  
  if (input === '0918') {
    // 直接归零，进度 100%
    countdown.value = { hours: 0, minutes: 0, seconds: 0, milliseconds: 0 }
    targetTime.value = new Date()
    // 设置开始时间为 48 小时前，让进度变成 100%
    startTime.value = new Date(Date.now() - totalDuration)
    if (timerInterval.value) clearInterval(timerInterval.value)
    closeModal()
  } else if (input === '1120') {
    // 恢复倒计时
    initCountdown()
    closeModal()
  } else {
    alert('输入错误哦～请输入正确的生日密码！❤️')
  }
}

// 总时长 48 小时（毫秒）
const totalDuration = 48 * 60 * 60 * 1000

// 雪花效果
const snowflakes = ref([])

// 进度百分比
const progressPercent = computed(() => {
  if (!startTime.value || !targetTime.value) return 0
  const now = new Date()
  const elapsed = now.getTime() - startTime.value.getTime()
  const percent = (elapsed / totalDuration) * 100
  return Math.min(100, Math.max(0, percent))
})

// 进度文字
const progressText = computed(() => {
  if (progressPercent.value >= 100) return '已经成功解冻'
  return '解冻进度'
})

// 图片路径
const currentImage = computed(() => {
  if (progressPercent.value >= 100) return imageGif
  return imageWebp
})

// 底部文字
const bottomText = computed(() => {
  if (progressPercent.value >= 100) return '谢谢小如，永远爱你＼(＠＾０＾＠)/♪'
  return '度秒如年，跪求解冻 (╥﹏╥)'
})

// 剩余时间
const remainingTime = computed(() => {
  if (!targetTime.value) return { hours: 0, minutes: 0, seconds: 0 }
  const now = new Date()
  const diff = targetTime.value.getTime() - now.getTime()
  if (diff <= 0) return { hours: 0, minutes: 0, seconds: 0 }
  return {
    hours: Math.floor(diff / 3600000),
    minutes: Math.floor((diff % 3600000) / 60000),
    seconds: Math.floor((diff % 60000) / 1000)
  }
})

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
// 开始时间 = 目标时间 - 48 小时
const initCountdown = () => {
  // 使用本地时间设置，避免时区问题
  targetTime.value = new Date(2026, 2, 15, 12, 27, 0) // 月份从 0 开始，2 代表 3 月
  startTime.value = new Date(targetTime.value.getTime() - totalDuration)
  
  if (timerInterval.value) clearInterval(timerInterval.value)
  timerInterval.value = setInterval(updateCountdown, 10)
  
  // 启动脉冲动画
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
    
    <!-- 心电图进度条区域 -->
    <div class="ecg-container">
      <!-- 左边的心 -->
      <div class="heart heart-left">
        <svg viewBox="0 0 24 24" fill="currentColor">
          <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/>
        </svg>
      </div>
      
      <!-- 进度条线条 -->
      <div class="progress-line">
        <!-- 灰色背景轨道 -->
        <div class="progress-track">
          <div class="progress-bg"></div>
        </div>
        <!-- 红色进度条 -->
        <div class="progress-fill" :style="{ width: progressPercent + '%' }">
          <!-- 脉冲光点 -->
          <div class="progress-pulse" :style="{ left: pulsePosition * 100 + '%' }"></div>
        </div>
        <!-- 心电图波形 -->
        <svg class="ecg-wave" viewBox="0 0 500 100" preserveAspectRatio="none">
          <defs>
            <linearGradient id="ecgGradient" x1="0%" y1="0%" x2="100%" y2="0%">
              <stop :offset="progressPercent + '%'" style="stop-color:#ff0040;stop-opacity:1" />
              <stop :offset="progressPercent + '%'" style="stop-color:#ff0040;stop-opacity:0" />
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
      <div class="heart heart-right" :class="{ active: progressPercent >= 100 }">
        <svg viewBox="0 0 24 24" fill="currentColor">
          <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/>
        </svg>
      </div>
    </div>
    
    <!-- 进度百分比显示 -->
    <div class="progress-text">
      <span class="progress-number">{{ progressPercent.toFixed(2) }}%</span>
      <span class="progress-label">{{ progressText }}</span>
    </div>
    
    <!-- 倒计时区域 -->
    <div class="countdown-container">
      <div class="countdown-title">冷静时间还剩：</div>
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
    
    <!-- 图片区域 -->
    <div class="image-container">
      <img :src="currentImage" alt="image" class="bottom-image" />
    </div>
    <div class="bottom-text">{{ bottomText }}</div>
    
    <!-- 一键解冻按钮 -->
    <button class="unfreeze-btn" @click="openModal">一键解冻</button>
    
    <!-- 弹窗 -->
    <div v-if="showModal" class="modal-overlay" @click.self="closeModal">
      <div class="modal-content">
        <div class="modal-title">请输入您的生日❤️</div>
        <div class="modal-hint">输入格式参考：1 月 1 日输入 0101</div>
        <input 
          v-model="birthdayInput" 
          type="text" 
          class="modal-input" 
          placeholder="请输入 4 位数字"
          maxlength="4"
          @keyup.enter="handleUnfreeze"
        />
        <div class="modal-actions">
          <button class="modal-cancel" @click="closeModal">取消</button>
          <button class="modal-confirm" @click="handleUnfreeze">同意解冻</button>
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
  margin-bottom: 40px;
  position: relative;
  z-index: 1;
}

.heart {
  width: 100px;
  height: 100px;
  color: #4a4a4a;
  animation: heartbeat 1s ease-in-out infinite;
  filter: drop-shadow(0 0 15px rgba(74, 74, 74, 0.5));
  flex-shrink: 0;
  transition: all 0.3s ease;
}

.heart-left {
  animation-delay: 0s;
  color: #ff0040;
  filter: drop-shadow(0 0 25px rgba(255, 0, 64, 0.8)) drop-shadow(0 0 50px rgba(255, 0, 64, 0.4));
}

.heart-right {
  animation-delay: 0.5s;
}

.heart-right.active {
  color: #ff0040;
  filter: drop-shadow(0 0 25px rgba(255, 0, 64, 0.8)) drop-shadow(0 0 50px rgba(255, 0, 64, 0.4));
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

.progress-line {
  position: relative;
  width: 400px;
  height: 100px;
}

.progress-track {
  position: absolute;
  top: 50%;
  left: 0;
  right: 0;
  height: 8px;
  transform: translateY(-50%);
  border-radius: 4px;
  overflow: hidden;
  background: rgba(255, 255, 255, 0.1);
}

.progress-bg {
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, #4a4a4a 0%, #3a3a3a 100%);
}

.progress-fill {
  position: absolute;
  top: 50%;
  left: 0;
  height: 8px;
  transform: translateY(-50%);
  background: linear-gradient(90deg, #ff0040 0%, #ff4466 50%, #ff0040 100%);
  border-radius: 4px;
  box-shadow: 
    0 0 20px rgba(255, 0, 64, 0.6),
    0 0 40px rgba(255, 0, 64, 0.4),
    0 0 60px rgba(255, 0, 64, 0.2);
  transition: width 0.1s linear;
  overflow: visible;
}

.progress-pulse {
  position: absolute;
  top: 50%;
  width: 30px;
  height: 12px;
  background: linear-gradient(90deg, transparent, #ffffff, #ff0040, #ffffff, transparent);
  transform: translateY(-50%);
  border-radius: 3px;
  box-shadow: 
    0 0 20px #ffffff,
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
  stroke: url(#ecgGradient);
  stroke-width: 3;
  stroke-linecap: round;
  stroke-linejoin: round;
  filter: url(#glow);
  opacity: 0.9;
}

.progress-text {
  text-align: center;
  margin-bottom: 30px;
  z-index: 1;
}

.progress-number {
  font-size: 32px;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.5);
  font-family: 'Consolas', 'Monaco', monospace;
  letter-spacing: 2px;
}

.progress-label {
  display: block;
  font-size: 14px;
  color: rgba(255, 255, 255, 0.6);
  margin-top: 5px;
  text-transform: uppercase;
  letter-spacing: 2px;
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

.image-container {
  margin-top: 40px;
  display: flex;
  justify-content: center;
  z-index: 1;
}

.bottom-image {
  max-width: 150px;
  max-height: 150px;
  width: auto;
  height: auto;
  border-radius: 12px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.bottom-text {
  margin-top: 15px;
  font-size: 14px;
  color: rgba(255, 255, 255, 0.6);
  text-align: center;
  letter-spacing: 1px;
}

.unfreeze-btn {
  margin-top: 20px;
  padding: 12px 40px;
  font-size: 16px;
  font-weight: 600;
  color: #ffffff;
  background: linear-gradient(135deg, #ff0040 0%, #ff4466 100%);
  border: none;
  border-radius: 25px;
  cursor: pointer;
  box-shadow: 0 4px 20px rgba(255, 0, 64, 0.4);
  transition: all 0.3s ease;
  letter-spacing: 2px;
}

.unfreeze-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 30px rgba(255, 0, 64, 0.6);
}

.unfreeze-btn:active {
  transform: translateY(0);
}

/* 弹窗样式 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  backdrop-filter: blur(5px);
}

.modal-content {
  background: linear-gradient(135deg, #1e0a3c 0%, #2d1b4e 100%);
  border-radius: 20px;
  padding: 30px 40px;
  max-width: 400px;
  width: 90%;
  border: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
}

.modal-title {
  font-size: 20px;
  color: #ffffff;
  text-align: center;
  margin-bottom: 10px;
  font-weight: 600;
}

.modal-hint {
  font-size: 13px;
  color: rgba(255, 255, 255, 0.5);
  text-align: center;
  margin-bottom: 20px;
}

.modal-input {
  width: 100%;
  padding: 15px 20px;
  font-size: 18px;
  text-align: center;
  letter-spacing: 5px;
  border: 2px solid rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  background: rgba(255, 255, 255, 0.05);
  color: #ffffff;
  margin-bottom: 20px;
  outline: none;
  transition: border-color 0.3s ease;
}

.modal-input:focus {
  border-color: #ff0040;
}

.modal-input::placeholder {
  color: rgba(255, 255, 255, 0.3);
  letter-spacing: 1px;
}

.modal-actions {
  display: flex;
  gap: 15px;
  justify-content: center;
}

.modal-cancel,
.modal-confirm {
  padding: 12px 30px;
  font-size: 15px;
  font-weight: 600;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  border: none;
}

.modal-cancel {
  background: rgba(255, 255, 255, 0.1);
  color: rgba(255, 255, 255, 0.8);
}

.modal-cancel:hover {
  background: rgba(255, 255, 255, 0.2);
}

.modal-confirm {
  background: linear-gradient(135deg, #ff0040 0%, #ff4466 100%);
  color: #ffffff;
  box-shadow: 0 4px 15px rgba(255, 0, 64, 0.4);
}

.modal-confirm:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 25px rgba(255, 0, 64, 0.6);
}

@media (max-width: 768px) {
  .container {
    padding: 10px;
  }
  
  .ecg-container {
    gap: 10px;
    margin-bottom: 30px;
    width: 100%;
    max-width: 400px;
  }
  
  .heart {
    width: 50px;
    height: 50px;
    flex-shrink: 0;
  }
  
  .progress-line {
    width: 100%;
    flex-grow: 1;
    height: 60px;
    min-width: 80px;
  }
  
  .progress-track {
    height: 6px;
  }
  
  .progress-fill {
    height: 6px;
  }
  
  .progress-pulse {
    width: 20px;
    height: 8px;
  }
  
  .progress-number {
    font-size: 36px;
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
  
  .image-container {
    margin-top: 25px;
  }
  
  .bottom-image {
    max-width: 120px;
    max-height: 120px;
  }
  
  .bottom-text {
    font-size: 13px;
    margin-top: 12px;
  }
  
  .unfreeze-btn {
    padding: 10px 30px;
    font-size: 14px;
    margin-top: 15px;
  }
  
  .modal-content {
    padding: 25px 30px;
  }
  
  .modal-title {
    font-size: 18px;
  }
  
  .modal-input {
    font-size: 16px;
    padding: 12px 15px;
  }
}

@media (max-width: 480px) {
  .ecg-container {
    gap: 5px;
    width: 100%;
    max-width: 320px;
  }
  
  .heart {
    width: 40px;
    height: 40px;
    flex-shrink: 0;
  }
  
  .progress-line {
    width: 100%;
    flex-grow: 1;
    min-width: 60px;
  }
  
  .progress-number {
    font-size: 28px;
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
  
  .image-container {
    margin-top: 20px;
  }
  
  .bottom-image {
    max-width: 90px;
    max-height: 90px;
  }
  
  .bottom-text {
    font-size: 12px;
    margin-top: 10px;
  }
  
  .unfreeze-btn {
    padding: 10px 25px;
    font-size: 14px;
    margin-top: 12px;
  }
  
  .modal-content {
    padding: 20px 25px;
    width: 85%;
  }
  
  .modal-title {
    font-size: 16px;
  }
  
  .modal-hint {
    font-size: 12px;
  }
  
  .modal-input {
    font-size: 16px;
    padding: 12px 15px;
  }
  
  .modal-cancel,
  .modal-confirm {
    padding: 10px 20px;
    font-size: 14px;
  }
}
</style>
