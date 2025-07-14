<script setup lang="ts">
import { gsap } from 'gsap'
import { onMounted, onUnmounted, ref } from 'vue'

// 组件根元素引用
const sectionRef = ref<HTMLElement>()
const featuresContainerRef = ref<HTMLElement>()

// 动画状态
const hasAnimated = ref(false)

// 功能特性数据
const features = [
  {
    title: "Rolldown-powered engine",
    progress: 100
  },
  {
    title: "Optimized compilation pipeline", 
    progress: 85
  },
  {
    title: "Parallel processing capabilities",
    progress: 80
  },
  {
    title: "Smart caching mechanisms",
    progress: 75
  },
  {
    title: "Efficient resource bundling",
    progress: 80
  }
]

// 创建模板引用
let tsupBarRef: HTMLElement | undefined
let tsdownBarRef: HTMLElement | undefined
let tsupTimeRef: HTMLElement | undefined
let tsdownTimeRef: HTMLElement | undefined

// Intersection Observer
let observer: IntersectionObserver | null = null

// 主时间轴
let mainTimeline: gsap.core.Timeline | null = null

// 动画函数
const runAnimation = async () => {
  if (tsupBarRef && tsdownBarRef && tsupTimeRef && tsdownTimeRef && featuresContainerRef.value) {
    // 创建主时间轴
    mainTimeline = gsap.timeline()
    
    // 确保所有引用都存在后进行类型断言
    const tsupBar = tsupBarRef as HTMLElement
    const tsdownBar = tsdownBarRef as HTMLElement
    const tsupTime = tsupTimeRef as HTMLElement
    const tsdownTime = tsdownTimeRef as HTMLElement
    
      // 获取特性元素
  const featureElements = featuresContainerRef.value.querySelectorAll('.feature-item')
    const rightContainer = featuresContainerRef.value.parentElement
    
    // 重新设置初始状态 - 确保所有元素都是隐藏的
    gsap.set([tsupBar, tsdownBar], {
      width: "0%",
      transformOrigin: "left center",
      boxShadow: "0 0 0 rgba(75, 85, 99, 0.4)"
    })
    gsap.set([tsupTime, tsdownTime], {
      innerText: "0.0s"
    })
    gsap.set(featureElements, {
      opacity: 0,
      y: 40,
      scale: 0.8
    })
    gsap.set(rightContainer, {
      opacity: 0,
      x: 50
    })
    
    // 重新设计的动画序列 - 更有序和流畅
    mainTimeline
      // 阶段1：右侧容器整体渐入
      .to(rightContainer, {
        opacity: 1,
        x: 0,
        duration: 0.8,
        ease: "power2.out"
      })
      
      // 阶段2：延迟0.3秒后，同时开始所有动画
      .to(tsdownBar, {
        width: "50%",
        duration: 2.1,
        ease: "power2.inOut",
        boxShadow: "0 0 25px rgba(251, 191, 36, 0.8)"
      }, "start+=0.3")
      
      // 同时开始tsup动画
      .to(tsupBar, {
        width: "100%",
        duration: 4.2,
        ease: "power2.inOut",
        boxShadow: "0 0 25px rgba(75, 85, 99, 0.8)"
      }, "start+=0.3")
      
      // tsdown数字动画与柱状图同步
      .to(tsdownTime, {
        innerText: "2.1s",
        duration: 2.1,
        ease: "none",
        snap: { innerText: 0.1 },
        onUpdate: function() {
          gsap.to(tsdownTime, {
            scale: 1.05,
            duration: 0.1,
            yoyo: true,
            repeat: 1
          })
        }
      }, "start+=0.3")
      
      // tsup数字动画与其柱状图同步
      .to(tsupTime, {
        innerText: "4.2s",
        duration: 4.2,
        ease: "none",
        snap: { innerText: 0.1 },
        onUpdate: function() {
          gsap.to(tsupTime, {
            scale: 1.05,
            duration: 0.1,
            yoyo: true,
            repeat: 1
          })
        }
      }, "start+=0.3")
      
      // 右侧特性动画也同时开始 - 一次性全部出现
      .to(featureElements, {
        opacity: 1,
        y: 0,
        scale: 1,
        duration: 0.8,
        ease: "back.out(1.2)"
      }, "start+=0.3")
      

      
              // 阶段4：最终高亮效果
      .to([tsupBar, tsdownBar], {
        boxShadow: "0 0 40px rgba(255, 255, 255, 0.3)",
        duration: 0.3,
        yoyo: true,
        repeat: 1
      }, "+=0.3")
  }
}

// 移除不需要的函数
// const animateAllProgressBars = async () => { ... }
// const scrollToElement = (element: Element) => { ... }

// 设置引用的函数
const setTsupBarRef = (el: any) => {
  if (el) {
    tsupBarRef = el as HTMLElement
    checkAndRun()
  }
}

const setTsdownBarRef = (el: any) => {
  if (el) {
    tsdownBarRef = el as HTMLElement
    checkAndRun()
  }
}

const setTsupTimeRef = (el: any) => {
  if (el) {
    tsupTimeRef = el as HTMLElement
    checkAndRun()
  }
}

const setTsdownTimeRef = (el: any) => {
  if (el) {
    tsdownTimeRef = el as HTMLElement
    checkAndRun()
  }
}

const checkAndRun = () => {
  if (tsupBarRef && tsdownBarRef && tsupTimeRef && tsdownTimeRef && hasAnimated.value) {
    runAnimation()
  }
}

// 设置Intersection Observer
const setupIntersectionObserver = () => {
  if (!sectionRef.value) return
  
  observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting && !hasAnimated.value) {
          hasAnimated.value = true
          // 延迟一点时间让滚动稳定
          setTimeout(() => {
            checkAndRun()
          }, 300)
        }
      })
    },
    {
      threshold: 0.3,
      rootMargin: '-50px'
    }
  )
  
  observer.observe(sectionRef.value)
}

onMounted(() => {
  setupIntersectionObserver()
})

onUnmounted(() => {
  if (observer) {
    observer.disconnect()
  }
  if (mainTimeline) {
    mainTimeline.kill()
  }
})
</script>

<template>
  <div ref="sectionRef" class="min-h-screen flex justify-center">
    <div class="w-full max-w-7xl px-8">
      <!-- 左右布局容器 -->
      <div class="flex gap-16 min-h-screen">
        <!-- 左侧：性能对比图表 -->
        <div class="flex-1 flex flex-col justify-center">
          <!-- 标题 -->
          <div class="mb-12">
            <h2 class="text-5xl font-bold text-white mb-4">Build Speed Comparison</h2>
            <p class="text-gray-400 text-xl">Standard TypeScript Project Build Time</p>
          </div>

          <!-- 横向柱状图容器 -->
          <div class="space-y-12">
            <!-- Tsdown 横向柱子 -->
            <div class="flex items-center gap-8">
              <div class="w-28 text-right">
                <div class="text-2xl font-bold text-white mb-2">Tsdown</div>
                <div 
                  :ref="setTsdownTimeRef"
                  class="text-xl text-yellow-400 font-mono font-bold"
                >
                  0.0s
                </div>
              </div>
              <div class="flex-1 h-14 bg-gray-800 rounded-xl relative overflow-hidden shadow-lg">
                <div 
                  :ref="setTsdownBarRef"
                  class="absolute left-0 top-0 h-full bg-gradient-to-r from-yellow-300 via-yellow-400 to-orange-500 rounded-xl"
                  style="width: 0%"
                ></div>
              </div>
            </div>

            <!-- Tsup 横向柱子 -->
            <div class="flex items-center gap-8">
              <div class="w-28 text-right">
                <div class="text-2xl font-bold text-white mb-2">Tsup</div>
                <div 
                  :ref="setTsupTimeRef"
                  class="text-xl text-gray-400 font-mono font-bold"
                >
                  0.0s
                </div>
              </div>
              <div class="flex-1 h-14 bg-gray-800 rounded-xl relative overflow-hidden shadow-lg">
                <div 
                  :ref="setTsupBarRef"
                  class="absolute left-0 top-0 h-full bg-gradient-to-r from-gray-600 via-gray-700 to-gray-900 rounded-xl"
                  style="width: 0%"
                ></div>
              </div>
            </div>
          </div>
        </div>

        <!-- 右侧：功能特性列表 -->
        <div class="flex-1 flex flex-col justify-center">
          <div class="backdrop-blur-xl bg-gradient-to-br from-gray-800/20 via-gray-700/10 to-gray-600/15 rounded-2xl p-8 border border-white/10 shadow-2xl relative overflow-hidden">
            <!-- 玻璃磨砂效果的内层光晕 -->
            <div class="absolute inset-0 rounded-2xl bg-gradient-to-br from-white/8 via-gray-400/4 to-white/6 pointer-events-none"></div>
            <!-- 顶部高光 -->
            <div class="absolute top-0 left-0 right-0 h-px bg-gradient-to-r from-transparent via-white/15 to-transparent"></div>
            <!-- 内容容器 -->
            <div class="relative z-10">
              <h3 class="text-4xl font-bold text-white mb-8 whitespace-nowrap">Performance Advantages</h3>
              
              <div ref="featuresContainerRef" class="space-y-4">
                <!-- 功能项 -->
                <div 
                  v-for="(feature, index) in features" 
                  :key="index"
                  class="feature-item flex items-center gap-4 group cursor-pointer"
                >
                  <div class="w-8 h-8 rounded-full bg-green-500 flex items-center justify-center flex-shrink-0 shadow-lg">
                    <svg class="w-5 h-5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path>
                    </svg>
                  </div>
                  <div class="flex-1 flex flex-col justify-center">
                    <div class="text-2xl text-white font-medium mb-1">{{ feature.title }}</div>
                    <div class="h-0.5 bg-gradient-to-r from-transparent via-white to-transparent opacity-30"></div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* 初始隐藏状态 - 防止页面加载时闪烁 */
.feature-item {
  opacity: 0;
  transform: translateY(40px) scale(0.8);
  padding: 0.75rem;
  border-radius: 0.75rem;
  transition: all 0.3s ease;
  min-height: 3.5rem; /* 确保有足够的高度 */
}



.backdrop-blur-sm {
  opacity: 0;
  transform: translateX(50px);
}

/* 功能项悬停效果 */
.feature-item:hover {
  transform: translateX(8px);
  transition: transform 0.3s ease;
}

.feature-item:hover .bg-green-500 {
  background: linear-gradient(135deg, #10b981, #059669);
  box-shadow: 0 4px 12px rgba(16, 185, 129, 0.4);
}



.feature-item:hover {
  background: rgba(31, 41, 55, 0.3);
  backdrop-filter: blur(8px);
}

/* 柱状图初始状态 */
.flex-1 .bg-gradient-to-r {
  width: 0%;
  box-shadow: 0 0 0 rgba(75, 85, 99, 0.4);
}
</style>
