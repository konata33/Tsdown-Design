<script setup lang="ts">
</script>

<template>
    <div class="flex items-start pt-20 min-h-[80vh]">

        <div class="flex-1 pl-25">
          <h1 class="text-super-large font-bold mb-6 tsdown-gradient">Tsdown</h1>
          <p class="text-lg mb-8 rolldown-gradient">Powered by Rolldown</p>
          
                      <div class="space-y-4 mb-8 text-xl">
              <div class="feature-card">
                <div class="feature-icon diamond bg-yellow-400"></div>
                <p class="text-gray-100"><span class="text-yellow-400 font-semibold">2x faster</span> than tsup for standard builds</p>
              </div>
              <div class="feature-card">
                <div class="feature-icon diamond bg-blue-400"></div>
                <p class="text-gray-100"><span class="text-blue-400 font-semibold">8x faster</span> TypeScript declaration generation</p>
              </div>
              <div class="feature-card">
                <div class="feature-icon diamond bg-green-400"></div>
                <p class="text-gray-100">Auto-generating package exports</p>
              </div>
              <div class="feature-card">
                <div class="feature-icon diamond bg-purple-400"></div>
                <p class="text-gray-100">Tree shaking & source maps included</p>
              </div>
            </div>
            
          
          <!-- 行动按钮 -->
          <div class="flex gap-4">
            <button class="px-6 py-3 bg-gradient-to-r from-blue-500 to-blue-600 text-white rounded-lg font-medium hover:from-blue-600 hover:to-blue-700 transition-all duration-200 shadow-lg">
              Get Started
            </button>
            <button class="px-6 py-3 border border-gray-500 text-gray-300 rounded-lg font-medium hover:border-gray-400 hover:text-white transition-all duration-200">
              View Docs
            </button>
          </div>
        </div>
        
        <div class="flex-1 ">
          <div class="flex items-center justify-center relative">
            <!-- 背景条纹装饰 -->
            <div class="absolute inset-0 overflow-hidden">
              <div class="stripe-background">
                <div class="stripe stripe-5"></div>
                <div class="stripe stripe-6"></div>
                <div class="stripe stripe-7"></div>
                <div class="stripe stripe-8"></div>
                <div class="stripe stripe-9"></div>
                <div class="stripe stripe-10"></div>
                <div class="stripe stripe-11"></div>
                <div class="stripe stripe-12"></div>
                <div class="stripe stripe-13"></div>
                <div class="stripe stripe-14"></div>
              </div>
            </div>
            
            <div class="relative z-10 ">
              <div class="absolute inset-0 bg-gradient-radial from-blue-400/4 via-yellow-400/6 via-orange-400/4 to-transparent rounded-full blur-lg scale-120 breathing-bg"></div>
              
              <img 
                src="/Logo.svg" 
                class="relative w-120 h-120 object-contain drop-shadow-lg glow-effect"
              />
            </div>
          </div>
        </div>
      </div>
</template>

<style scoped>
/* 超大字体大小 */
.text-super-large {
  font-size: 8rem; /* 相当于 text-8xl 以上 */
  line-height: 1.1;
}

/* 在较小屏幕上调整字体大小 */
@media (max-width: 768px) {
  .text-super-large {
    font-size: 4rem;
  }
}

@media (max-width: 480px) {
  .text-super-large {
    font-size: 3rem;
  }
}

/* Tsdown 渐变文字效果 */
.tsdown-gradient {
  background: -webkit-linear-gradient(90deg, #2563eb, #3b82f6, #fbbf24);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

/* Rolldown 渐变文字效果 */
.rolldown-gradient {
  background: -webkit-linear-gradient(90deg, #ff5d13, #f0db4f);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  font-weight: 500;
}

/* 自定义光晕效果 */
.glow-effect {
  filter: drop-shadow(0 0 12px rgba(251, 191, 36, 0.15)) 
          drop-shadow(0 0 24px rgba(249, 115, 22, 0.1))
          drop-shadow(0 0 48px rgba(255, 193, 7, 0.08));
  animation: breathingGlow 3s ease-in-out infinite;
}

/* 径向渐变背景 */
.bg-gradient-radial {
  background: radial-gradient(circle, var(--tw-gradient-stops));
}

/* 呼吸灯动画 */
@keyframes breathingGlow {
  0%, 100% {
    filter: drop-shadow(0 0 10px rgba(59, 130, 246, 0.12)) 
            drop-shadow(0 0 20px rgba(251, 191, 36, 0.15))
            drop-shadow(0 0 40px rgba(147, 197, 253, 0.08));
  }
  50% {
    filter: drop-shadow(0 0 16px rgba(59, 130, 246, 0.18)) 
            drop-shadow(0 0 32px rgba(251, 191, 36, 0.22))
            drop-shadow(0 0 60px rgba(147, 197, 253, 0.12));
  }
}

/* 背景呼吸动画 */
.breathing-bg {
  animation: breathingBackground 3s ease-in-out infinite;
}

@keyframes breathingBackground {
  0%, 100% {
    opacity: 0.8;
    transform: scale(1.2);
  }
  50% {
    opacity: 1;
    transform: scale(1.35);
  }
}

/* 特性卡片毛玻璃效果 */
.feature-card {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 1.25rem 1.5rem;
  background: rgb(84 79 79 / 5%);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border: 1px solid rgb(84 79 79 / 5%);
  border-radius: 1rem;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

/* 特性卡片悬停效果 */
.feature-card:hover {
  background: rgb(84 79 79 / 8%);
  border-color: rgb(84 79 79 / 10%);
  transform: translateY(-2px);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
}

/* 特性卡片伪元素光效 */
.feature-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.03),
    rgba(255, 255, 255, 0.05),
    rgba(255, 255, 255, 0.03),
    transparent
  );
  transition: left 0.8s ease, opacity 0.3s ease;
  opacity: 0;
}

.feature-card:hover::before {
  left: 100%;
  opacity: 1;
}

/* 特性图标样式 */
.feature-icon {
  flex-shrink: 0;
  width: 10px;
  height: 10px;
  transition: transform 0.3s ease;
}

/* 菱形 */
.diamond {
  transform: rotate(45deg);
  border-radius: 2px;
}

/* 小箭头 */
.arrow-right {
  width: 0;
  height: 0;
  border-left: 6px solid currentColor;
  border-top: 4px solid transparent;
  border-bottom: 4px solid transparent;
  background: none !important;
}

/* 六边形 */
.hexagon {
  width: 12px;
  height: 10px;
  position: relative;
  border-radius: 2px;
}

.hexagon::before,
.hexagon::after {
  content: '';
  position: absolute;
  width: 0;
  height: 0;
  border-left: 6px solid transparent;
  border-right: 6px solid transparent;
}

.hexagon::before {
  top: -3px;
  border-bottom: 3px solid currentColor;
}

.hexagon::after {
  bottom: -3px;
  border-top: 3px solid currentColor;
}

/* 勾选标记 */
.checkmark {
  width: 12px;
  height: 12px;
  position: relative;
  font-size: 10px;
  font-weight: bold;
}

.checkmark::after {
  content: '✓';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.feature-card:hover .feature-icon {
  transform: rotate(45deg) scale(1.1);
}

/* 背景条纹装饰 */
.stripe-background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  opacity: 0.8;
}


.stripe {
  position: absolute;
  height: 6px;
  background: linear-gradient(
    135deg,
    rgba(15, 15, 20, 0.4),
    rgba(35, 35, 40, 0.3),
    rgba(55, 55, 60, 0.2),
    rgba(35, 35, 40, 0.3),
    rgba(15, 15, 20, 0.4)
  );
  border-radius: 3px;
  border: 1px solid rgba(255, 255, 255, 0.015);
  transform: rotate(-25deg);
  transform-origin: center;
  filter: blur(0.5px);
  backdrop-filter: blur(2px);
  -webkit-backdrop-filter: blur(2px);
}

.stripe-1 { top: 15%; right: 1%; width: 100%; }
.stripe-2 { top: 20%; right: 1%; width: 100%; }
.stripe-3 { top: 25%; right: 1%; width: 100%; }
.stripe-4 { top: 30%; right: 1%; width: 100%; }
.stripe-5 { top: 35%; right: 1%; width: 100%; }
.stripe-6 { top: 40%; right: 1%; width: 100%; }
.stripe-7 { top: 45%; right: 1%; width: 100%; }
.stripe-8 { top: 50%; right: 1%; width: 100%; }
.stripe-9 { top: 55%; right: 1%; width: 100%; }
.stripe-10 { top: 60%; right: 1%; width: 100%; }
.stripe-11 { top: 65%; right: 1%; width: 100%; }
.stripe-12 { top: 70%; right: 1%; width: 100%; }
.stripe-13 { top: 75%; right: 1%; width: 100%; }
.stripe-14 { top: 80%; right: 1%; width: 100%; }
</style>
