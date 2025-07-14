<script setup lang="ts">
import { gsap } from 'gsap'
import { nextTick, onMounted, onUnmounted, ref } from 'vue'

// 动画状态
const isAnimating = ref(false)
const hasAnimated = ref(false) // 防止重复播放
const currentPhase = ref(0)
const overallProgress = ref(0)

// 组件根元素引用
const sectionRef = ref<HTMLElement>()
const titleRef = ref<HTMLElement>()
const cardsRef = ref<HTMLElement>()
const statsRef = ref<HTMLElement>()

// 数据状态
const configLines = ref<string[]>([])
const scannedFiles = ref<string[]>([])
const generatedExports = ref<string[]>([])
const buildStats = ref({
  filesScanned: 0,
  exportsGenerated: 0,
  timeTaken: 0
})

// 滚动容器引用
const configContainer = ref<HTMLElement>()
const scanContainer = ref<HTMLElement>()
const generateContainer = ref<HTMLElement>()

// Intersection Observer
let observer: IntersectionObserver | null = null
let entranceTimeline: gsap.core.Timeline | null = null

// 项目文件
const projectFiles = [
  { path: 'src/index.ts', exports: ['default', 'createApp'] },
  { path: 'src/utils.ts', exports: ['formatDate', 'debounce'] },
  { path: 'src/components/Button.tsx', exports: ['Button'] },
  { path: 'src/components/Modal.tsx', exports: ['Modal'] },
  { path: 'src/hooks/useStore.ts', exports: ['useStore'] },
  { path: 'src/types.ts', exports: ['ApiResponse', 'User'] }
]

// 配置代码
const configCode = [
  "import { defineConfig } from 'tsdown'",
  "",
  "export default defineConfig({",
  "  exports: {",
  "    all: true,",
  "    devExports: true,",
  "    subpaths: true",
  "  }",
  "})"
]

// 滚动到底部函数
const scrollToBottom = (container: HTMLElement | undefined) => {
  if (container) {
    container.scrollTop = container.scrollHeight
  }
}

// 创建入场动画
const createEntranceAnimation = () => {
  if (entranceTimeline) {
    entranceTimeline.kill()
  }
  
  if (!titleRef.value || !cardsRef.value) return
  
  entranceTimeline = gsap.timeline()
  
  // 设置初始状态
  gsap.set(titleRef.value, {
    opacity: 0,
    y: 50,
    visibility: "visible"
  })
  
  // 设置卡片的初始状态
  gsap.set(cardsRef.value.children, {
    opacity: 0,
    y: 50,
    visibility: "visible"
  })
  
  // 标题动画
  entranceTimeline.to(titleRef.value, {
    opacity: 1,
    y: 0,
    duration: 0.8,
    ease: "power2.out"
  })
  
  // 三个卡片的渐入动画
  entranceTimeline.to(cardsRef.value.children, {
    opacity: 1,
    y: 0,
    duration: 0.6,
    stagger: 0.2,
    ease: "power2.out"
  }, "-=0.3")
  
  return entranceTimeline
}

// 底部统计信息动画
const animateStats = () => {
  if (statsRef.value) {
    gsap.to(statsRef.value, {
      opacity: 1,
      y: 0,
      scale: 1,
      duration: 0.8,
      ease: "power2.out"
    })
  }
}

// 主动画函数
const startAnimation = async () => {
  if (isAnimating.value || hasAnimated.value) return
  
  // 启动入场动画（不等待完成）
  createEntranceAnimation()
  
  // 等待卡片开始出现
  await new Promise(resolve => setTimeout(resolve, 400))
  
  isAnimating.value = true
  hasAnimated.value = true
  
  // 重置状态
  configLines.value = []
  scannedFiles.value = []
  generatedExports.value = []
  currentPhase.value = 0
  overallProgress.value = 0
  buildStats.value = { filesScanned: 0, exportsGenerated: 0, timeTaken: 0 }
  
  // 三个阶段并行执行
  const exports = [
    '".": {',
    '  "import": "./dist/index.js",',
    '  "types": "./dist/index.d.ts"',
    '},',
    '"./utils": {',
    '  "import": "./dist/utils.js",',
    '  "types": "./dist/utils.d.ts"',
    '},',
    '"./components": {',
    '  "import": "./dist/components.js",',
    '  "types": "./dist/components.d.ts"',
    '},',
    '"./hooks": {',
    '  "import": "./dist/hooks.js",',
    '  "types": "./dist/hooks.d.ts"',
    '},',
    '"./types": {',
    '  "import": "./dist/types.js",',
    '  "types": "./dist/types.d.ts"',
    '}'
  ]
  
  // 阶段1：配置解析动画
  const configAnimation = async () => {
    await new Promise(resolve => setTimeout(resolve, 0)) // 立即开始
    currentPhase.value = Math.max(currentPhase.value, 1)
    for (let i = 0; i < configCode.length; i++) {
      configLines.value.push(configCode[i])
      await nextTick()
      scrollToBottom(configContainer.value)
      await new Promise(resolve => setTimeout(resolve, 80))
    }
  }
  
  // 阶段2：文件扫描动画
  const scanAnimation = async () => {
    await new Promise(resolve => setTimeout(resolve, 200)) // 稍微延迟开始
    currentPhase.value = Math.max(currentPhase.value, 2)
    for (let i = 0; i < projectFiles.length; i++) {
      scannedFiles.value.push(projectFiles[i].path)
      buildStats.value.filesScanned = i + 1
      await nextTick()
      scrollToBottom(scanContainer.value)
      await new Promise(resolve => setTimeout(resolve, 180))
    }
  }
  
  // 阶段3：导出生成动画
  const generateAnimation = async () => {
    await new Promise(resolve => setTimeout(resolve, 400)) // 再稍微延迟开始
    currentPhase.value = Math.max(currentPhase.value, 3)
    
    // 优化：批量添加和更快的速度
    for (let i = 0; i < exports.length; i++) {
      generatedExports.value.push(exports[i])
      
      // 每完成一个导出模块就增加计数 - 只在模块开始时计数
      if (exports[i].includes('".":') || exports[i].includes('"./')) {
        buildStats.value.exportsGenerated++
      }
      
      // 每3行滚动一次，减少频繁的DOM操作
      if (i % 3 === 0 || i === exports.length - 1) {
        await nextTick()
        scrollToBottom(generateContainer.value)
      }
      
      // 进一步减少延迟时间
      await new Promise(resolve => setTimeout(resolve, 25))
    }
  }
  
  // 进度更新动画
  const progressAnimation = () => {
    setTimeout(() => overallProgress.value = 40, 300)  // 0.3秒后40%
    setTimeout(() => overallProgress.value = 75, 800)  // 0.8秒后75%
    setTimeout(() => overallProgress.value = 95, 1400) // 1.4秒后95%
    // 在接近完成时就准备数据和阶段
    setTimeout(() => {
      buildStats.value.timeTaken = 1.2
      currentPhase.value = 4
      // 立即设置初始隐藏状态，避免闪烁
      nextTick(() => {
        if (statsRef.value) {
          gsap.set(statsRef.value, {
            opacity: 0,
            y: 30,
            scale: 0.9,
            visibility: "visible"
          })
        }
      })
    }, 1200) // 1.2秒后准备数据
  }
  
  // 启动进度更新
  progressAnimation()
  
  // 并行执行三个阶段和底部统计信息的准备
  const mainAnimations = Promise.all([
    configAnimation(),
    scanAnimation(),
    generateAnimation()
  ])
  
  // 在主动画接近完成时就开始底部统计信息的动画
  const statsAnimation = (async () => {
    await new Promise(resolve => setTimeout(resolve, 1300)) // 1.3秒后开始
    animateStats()
  })()
  
  // 等待主动画完成
  await mainAnimations
  
  // 完成进度
  overallProgress.value = 100
  
  isAnimating.value = false
}

// 设置Intersection Observer
const setupIntersectionObserver = () => {
  if (!sectionRef.value) return
  
  observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting && !hasAnimated.value) {
          // 延迟一点时间让滚动稳定
          setTimeout(() => {
            startAnimation()
          }, 300)
        }
      })
    },
    {
      threshold: 0.3, // 当30%的元素可见时触发
      rootMargin: '-50px' // 提前50px触发
    }
  )
  
  observer.observe(sectionRef.value)
}

onMounted(() => {
  // 使用nextTick确保DOM完全渲染后再设置初始状态
  nextTick(() => {
    // 立即设置初始隐藏状态，防止闪烁
    if (titleRef.value) {
      gsap.set(titleRef.value, {
        opacity: 0,
        y: 50,
        visibility: "visible"
      })
    }
    
    if (cardsRef.value) {
      gsap.set(cardsRef.value.children, {
        opacity: 0,
        y: 50,
        visibility: "visible"
      })
    }
    
    if (statsRef.value) {
      gsap.set(statsRef.value, {
        opacity: 0,
        y: 30,
        scale: 0.9,
        visibility: "visible"
      })
    }
    
    setupIntersectionObserver()
  })
})

onUnmounted(() => {
  if (observer) {
    observer.disconnect()
  }
  if (entranceTimeline) {
    entranceTimeline.kill()
  }
})
</script>

<template>
  <div ref="sectionRef" class="min-h-screen flex flex-col justify-center items-center py-16">
    <div class="w-full max-w-6xl px-8">
      <!-- 主标题 -->
      <div ref="titleRef" class="text-center mb-16">
        <h2 class="text-6xl font-bold mb-4 text-white">Package Exports</h2>
        <p class="text-2xl text-gray-400">Automatic generation workflow</p>
      </div>

      <!-- 中央仪表盘 -->
      <div class="relative">
        <!-- 三个阶段卡片 -->
        <div ref="cardsRef" class="grid grid-cols-3 gap-8 mb-12">
          <!-- 配置阶段 -->
          <div class="relative">
            <div :class="[
              'p-6 rounded-xl border-2 transition-all duration-500 backdrop-blur-sm h-80',
              currentPhase >= 1 ? 'border-blue-400 bg-blue-500/10' : 'border-gray-600 bg-gray-800/50'
            ]">
              <div class="flex items-center gap-3 mb-4">
                <div :class="[
                  'w-10 h-10 rounded-full flex items-center justify-center transition-all duration-300',
                  currentPhase >= 1 ? 'bg-blue-500' : 'bg-gray-600'
                ]">
                  <svg class="w-5 h-5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"></path>
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"></path>
                  </svg>
                </div>
                <h3 class="text-lg font-bold text-white">1. Configure</h3>
              </div>
              
              <div ref="configContainer" class="bg-gray-900 rounded-lg p-3 font-mono text-xs h-56 overflow-y-auto scroll-smooth">
                <div v-for="(line, index) in configLines" :key="index" class="text-gray-300 leading-relaxed break-words">
                  {{ line }}
                </div>
                <div v-if="currentPhase === 1 && configLines.length < configCode.length" class="animate-pulse bg-blue-400 w-2 h-4 inline-block"></div>
              </div>
            </div>
          </div>

          <!-- 扫描阶段 -->
          <div class="relative">
            <div :class="[
              'p-6 rounded-xl border-2 transition-all duration-500 backdrop-blur-sm h-80',
              currentPhase >= 2 ? 'border-yellow-400 bg-yellow-500/10' : 'border-gray-600 bg-gray-800/50'
            ]">
              <div class="flex items-center gap-3 mb-4">
                <div :class="[
                  'w-10 h-10 rounded-full flex items-center justify-center transition-all duration-300',
                  currentPhase >= 2 ? 'bg-yellow-500' : 'bg-gray-600'
                ]">
                  <svg class="w-5 h-5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path>
                  </svg>
                </div>
                <h3 class="text-lg font-bold text-white">2. Scan</h3>
              </div>
              
              <div ref="scanContainer" class="h-56 overflow-y-auto scroll-smooth">
                <div class="space-y-2">
                  <div v-for="(file, index) in scannedFiles" :key="index" 
                       class="flex items-center gap-2 p-2 bg-gray-800/50 rounded-lg animate-slideIn"
                       :style="{ animationDelay: `${index * 0.1}s` }">
                    <div class="w-2 h-2 bg-yellow-400 rounded-full animate-pulse flex-shrink-0"></div>
                    <span class="text-gray-300 font-mono text-xs truncate">{{ file }}</span>
                  </div>
                </div>
                
                <div v-if="currentPhase >= 2" class="mt-4 p-3 bg-gray-900 rounded-lg">
                  <div class="text-xs text-gray-400">Files scanned: <span class="text-yellow-400 font-bold">{{ buildStats.filesScanned }}</span></div>
                </div>
              </div>
            </div>
          </div>

          <!-- 生成阶段 -->
          <div class="relative">
            <div :class="[
              'p-6 rounded-xl border-2 transition-all duration-500 backdrop-blur-sm h-80',
              currentPhase >= 3 ? 'border-green-400 bg-green-500/10' : 'border-gray-600 bg-gray-800/50'
            ]">
              <div class="flex items-center gap-3 mb-4">
                <div :class="[
                  'w-10 h-10 rounded-full flex items-center justify-center transition-all duration-300',
                  currentPhase >= 3 ? 'bg-green-500' : 'bg-gray-600'
                ]">
                  <svg class="w-5 h-5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"></path>
                  </svg>
                </div>
                <h3 class="text-lg font-bold text-white">3. Generate</h3>
              </div>
              
              <div ref="generateContainer" class="bg-gray-900 rounded-lg p-3 font-mono text-xs h-56 overflow-y-auto scroll-smooth">
                <div class="text-gray-400 text-xs mb-2">"exports": {</div>
                <div v-for="(exportLine, index) in generatedExports" :key="index" 
                     class="text-gray-300 leading-relaxed animate-fadeIn break-all"
                     :style="{ animationDelay: `${index * 0.1}s` }">
                  <span class="pl-2">{{ exportLine }}</span>
                </div>
                <div v-if="currentPhase === 3 && generatedExports.length > 0" class="text-gray-400 text-xs mt-2">}</div>
                
                <div v-if="currentPhase >= 3" class="mt-4 p-2 bg-gray-800 rounded text-xs">
                  <div class="text-green-400">✓ {{ buildStats.exportsGenerated }} exports generated</div>
                  <div v-if="buildStats.timeTaken > 0" class="text-gray-400">⏱️ {{ buildStats.timeTaken }}s</div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- 底部统计信息 -->
        <div v-if="currentPhase >= 4" ref="statsRef" class="text-center">
          <div class="inline-flex items-center gap-8 bg-gray-800/50 backdrop-blur-sm rounded-2xl px-8 py-4 border border-gray-700">
            <div class="text-center">
              <div class="text-2xl font-bold text-yellow-400">{{ buildStats.filesScanned }}</div>
              <div class="text-sm text-gray-400">Files Scanned</div>
            </div>
            <div class="w-px h-8 bg-gray-600"></div>
            <div class="text-center">
              <div class="text-2xl font-bold text-yellow-400">{{ buildStats.exportsGenerated }}</div>
              <div class="text-sm text-gray-400">Exports Generated</div>
            </div>
            <div class="w-px h-8 bg-gray-600"></div>
            <div class="text-center">
              <div class="text-2xl font-bold text-yellow-400">{{ buildStats.timeTaken }}s</div>
              <div class="text-sm text-gray-400">Time Taken</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* 初始隐藏状态 - 防止页面加载时闪烁 */
[ref="titleRef"] {
  opacity: 0 !important;
  transform: translateY(50px) !important;
  visibility: hidden;
}

[ref="cardsRef"] > * {
  opacity: 0 !important;
  transform: translateY(50px) !important;
  visibility: hidden;
}

[ref="statsRef"] {
  opacity: 0 !important;
  transform: translateY(30px) scale(0.9) !important;
  visibility: hidden;
}

/* Tsdown 渐变文字效果 - 与IntroSection保持一致 */
.tsdown-gradient {
  background: -webkit-linear-gradient(90deg, #2563eb, #3b82f6, #fbbf24);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

/* 确保文字不会溢出 */
.bg-gray-900 {
  word-wrap: break-word;
  overflow-wrap: break-word;
  hyphens: auto;
}

/* 文件列表项样式 */
.truncate {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* 平滑滚动 */
.scroll-smooth {
  scroll-behavior: smooth;
}

/* 自定义动画 */
@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(-20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-slideIn {
  animation: slideIn 0.3s ease-out forwards;
}

.animate-fadeIn {
  animation: fadeIn 0.3s ease-out forwards;
}

/* 卡片悬停效果 */
.backdrop-blur-sm:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.3);
}

/* 阶段完成时的发光效果 */
.border-blue-400 {
  box-shadow: 0 0 20px rgba(59, 130, 246, 0.2);
}

.border-yellow-400 {
  box-shadow: 0 0 20px rgba(251, 191, 36, 0.2);
}

.border-green-400 {
  box-shadow: 0 0 20px rgba(34, 197, 94, 0.2);
}
</style> 

