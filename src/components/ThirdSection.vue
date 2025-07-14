<script setup lang="ts">
import { nextTick, onMounted, onUnmounted, ref } from 'vue'

// 组件根元素引用
const sectionRef = ref<HTMLElement>()

// 代码容器引用
const tsdownCodeContainer = ref<HTMLElement>()
const tsupCodeContainer = ref<HTMLElement>()

// 动画状态
const hasAnimated = ref(false)

// 模拟的TypeScript声明文件内容
const declarationCode = `export interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
}

export declare function createUser(data: Partial<User>): Promise<User>;

export declare function updateUser(id: number, data: Partial<User>): Promise<User>;

export declare function deleteUser(id: number): Promise<void>;

export declare class UserService {
  private users: User[];
  constructor();
  getAllUsers(): Promise<User[]>;
  getUserById(id: number): Promise<User | undefined>;
  createUser(userData: Partial<User>): Promise<User>;
  updateUser(id: number, userData: Partial<User>): Promise<User>;
  deleteUser(id: number): Promise<void>;
}

export declare type UserStatus = 'active' | 'inactive' | 'pending';

export declare interface UserConfig {
  maxUsers: number;
  defaultRole: string;
  enableNotifications: boolean;
}`

// 响应式状态
const tsdownCode = ref('')
const tsupCode = ref('')
const tsdownProgress = ref(0)
const tsupProgress = ref(0)
const isAnimating = ref(false)

// Intersection Observer
let observer: IntersectionObserver | null = null

// 滚动到底部函数
const scrollToBottom = (container: HTMLElement | undefined) => {
  if (container) {
    container.scrollTop = container.scrollHeight
  }
}

// 平滑滚动函数
const smoothScrollToBottom = (container: HTMLElement | undefined) => {
  if (container) {
    container.scrollTo({
      top: container.scrollHeight,
      behavior: 'smooth'
    })
  }
}

// 打字机动画函数（增强版，支持滚动）
const typewriter = (
  text: string,
  targetRef: { value: string },
  progressRef: { value: number },
  container: HTMLElement | undefined,
  speed: number = 50
) => {
  return new Promise<void>((resolve) => {
    let index = 0
    let lineCount = 0
    const timer = setInterval(async () => {
      if (index < text.length) {
        targetRef.value += text[index]
        progressRef.value = Math.round((index / text.length) * 100)
        
        // 检查是否添加了换行符，每几行滚动一次
        if (text[index] === '\n') {
          lineCount++
          if (lineCount % 3 === 0) { // 每3行滚动一次
            await nextTick()
            scrollToBottom(container)
          }
        }
        
        // 定期检查是否需要滚动（每50个字符）
        if (index % 50 === 0) {
          await nextTick()
          scrollToBottom(container)
        }
        
        index++
      } else {
        // 动画完成时最后滚动一次
        await nextTick()
        smoothScrollToBottom(container)
        clearInterval(timer)
        resolve()
      }
    }, speed)
  })
}

// 开始动画
const startAnimation = async () => {
  if (isAnimating.value || !hasAnimated.value) return
  
  isAnimating.value = true
  
  // 重置状态
  tsdownCode.value = ''
  tsupCode.value = ''
  tsdownProgress.value = 0
  tsupProgress.value = 0
  
  // 确保容器滚动到顶部
  if (tsdownCodeContainer.value) tsdownCodeContainer.value.scrollTop = 0
  if (tsupCodeContainer.value) tsupCodeContainer.value.scrollTop = 0
  
  // 同时开始两个动画，tsdown速度快4倍
  await Promise.all([
    typewriter(declarationCode, tsdownCode, tsdownProgress, tsdownCodeContainer.value, 15), // 快速
    typewriter(declarationCode, tsupCode, tsupProgress, tsupCodeContainer.value, 60)     // 中等速度
  ])
  
  isAnimating.value = false
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
  setupIntersectionObserver()
})

onUnmounted(() => {
  if (observer) {
    observer.disconnect()
  }
})
</script>

<template>
  <div ref="sectionRef" class="min-h-screen flex justify-center items-center bg-gray-950">
    <div class="w-full max-w-7xl px-8">
      <!-- 标题 -->
      <div class="text-center mb-12">
        <h2 class="text-5xl font-bold text-white mb-4">TypeScript Declaration Generation</h2>
        <p class="text-gray-400 text-xl">8x faster .d.ts file generation</p>
      </div>

      <!-- 左右对比代码编辑器 -->
      <div class="flex gap-8">
        <!-- Tsdown 编辑器 -->
        <div class="flex-1">
          <!-- Tsdown 品牌标识 -->
          <div class="mb-4 text-center">
            <div class="inline-flex items-center gap-3 bg-gradient-to-r from-yellow-400 to-orange-500 text-black px-6 py-3 rounded-xl font-bold text-xl shadow-lg">
              <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z"></path>
              </svg>
              <span>Tsdown</span>
            </div>
          </div>
          
          <div class="bg-gray-900 rounded-lg overflow-hidden shadow-2xl border-2 border-yellow-400/30">
            <!-- 编辑器头部 -->
            <div class="bg-gradient-to-r from-yellow-900/20 to-orange-900/20 px-4 py-3 border-b border-yellow-400/20">
              <div class="flex items-center justify-between">
                <div class="flex items-center gap-3">
                  <div class="flex gap-1.5">
                    <div class="w-3 h-3 bg-red-500 rounded-full"></div>
                    <div class="w-3 h-3 bg-yellow-500 rounded-full"></div>
                    <div class="w-3 h-3 bg-green-500 rounded-full"></div>
                  </div>
                  <div class="flex items-center gap-2">
                    <span class="text-yellow-400 font-semibold">index.d.ts</span>
                  </div>
                </div>
              </div>
            </div>
            
            <!-- 进度条 -->
            <div class="bg-gradient-to-r from-yellow-900/10 to-orange-900/10 px-4 py-3 border-b border-yellow-400/20">
              <div class="flex items-center gap-3">
                <span class="text-sm text-yellow-300">Progress:</span>
                <div class="flex-1 h-3 bg-gray-800 rounded-full overflow-hidden shadow-inner">
                  <div 
                    class="h-full bg-gradient-to-r from-yellow-400 via-orange-400 to-orange-500 rounded-full transition-all duration-200 shadow-lg"
                    :style="{ width: `${tsdownProgress}%` }"
                  ></div>
                </div>
                <span class="text-sm text-yellow-400 font-mono font-bold">{{ tsdownProgress }}%</span>
              </div>
            </div>
            
            <!-- 代码区域 -->
            <div ref="tsdownCodeContainer" class="p-4 bg-gray-900 font-mono text-sm leading-relaxed h-96 overflow-y-auto scroll-smooth">
              <pre class="text-gray-300 whitespace-pre-wrap">{{ tsdownCode }}<span class="animate-pulse bg-yellow-400 w-2 h-5 inline-block ml-1"></span></pre>
            </div>
          </div>
        </div>

        <!-- Tsup 编辑器 -->
        <div class="flex-1">
          <!-- Tsup 品牌标识 -->
          <div class="mb-4 text-center">
            <div class="inline-flex items-center gap-3 bg-gradient-to-r from-gray-600 to-gray-700 text-white px-6 py-3 rounded-xl font-bold text-xl shadow-lg">
              <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"></path>
              </svg>
              <span>Tsup</span>
            </div>
          </div>
          
          <div class="bg-gray-900 rounded-lg overflow-hidden shadow-2xl border-2 border-gray-600/30">
            <!-- 编辑器头部 -->
            <div class="bg-gradient-to-r from-gray-800/50 to-gray-700/50 px-4 py-3 border-b border-gray-600/20">
              <div class="flex items-center justify-between">
                <div class="flex items-center gap-3">
                  <div class="flex gap-1.5">
                    <div class="w-3 h-3 bg-red-500 rounded-full"></div>
                    <div class="w-3 h-3 bg-yellow-500 rounded-full"></div>
                    <div class="w-3 h-3 bg-green-500 rounded-full"></div>
                  </div>
                  <div class="flex items-center gap-2">
                    <span class="text-gray-400 font-semibold">index.d.ts</span>
                  </div>
                </div>
              </div>
            </div>
            
            <!-- 进度条 -->
            <div class="bg-gradient-to-r from-gray-800/20 to-gray-700/20 px-4 py-3 border-b border-gray-600/20">
              <div class="flex items-center gap-3">
                <span class="text-sm text-gray-300">Progress:</span>
                <div class="flex-1 h-3 bg-gray-800 rounded-full overflow-hidden shadow-inner">
                  <div 
                    class="h-full bg-gradient-to-r from-gray-500 via-gray-600 to-gray-700 rounded-full transition-all duration-200 shadow-lg"
                    :style="{ width: `${tsupProgress}%` }"
                  ></div>
                </div>
                <span class="text-sm text-gray-500 font-mono font-bold">{{ tsupProgress }}%</span>
              </div>
            </div>
            
            <!-- 代码区域 -->
            <div ref="tsupCodeContainer" class="p-4 bg-gray-900 font-mono text-sm leading-relaxed h-96 overflow-y-auto scroll-smooth">
              <pre class="text-gray-300 whitespace-pre-wrap">{{ tsupCode }}<span class="animate-pulse bg-gray-400 w-2 h-5 inline-block ml-1"></span></pre>
            </div>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<style scoped>
/* 语法高亮效果 */
.syntax-highlight {
  color: #e5e7eb;
}

.syntax-keyword {
  color: #f59e0b;
}

.syntax-type {
  color: #3b82f6;
}

.syntax-string {
  color: #10b981;
}

.syntax-comment {
  color: #6b7280;
  font-style: italic;
}

/* 添加一些动画效果 */
@keyframes glow-yellow {
  0%, 100% { 
    box-shadow: 0 0 10px rgba(251, 191, 36, 0.3);
  }
  50% { 
    box-shadow: 0 0 20px rgba(251, 191, 36, 0.5);
  }
}

@keyframes glow-gray {
  0%, 100% { 
    box-shadow: 0 0 10px rgba(156, 163, 175, 0.2);
  }
  50% { 
    box-shadow: 0 0 20px rgba(156, 163, 175, 0.4);
  }
}

/* 为品牌标识添加悬停效果 */
.flex-1:first-child .inline-flex:hover {
  animation: glow-yellow 1.5s ease-in-out infinite;
}

.flex-1:last-child .inline-flex:hover {
  animation: glow-gray 1.5s ease-in-out infinite;
}

/* 为编辑器容器添加微妙的阴影动画 */
.border-yellow-400\/30 {
  box-shadow: 0 0 30px rgba(251, 191, 36, 0.1);
}

.border-gray-600\/30 {
  box-shadow: 0 0 30px rgba(156, 163, 175, 0.1);
}

/* 添加进度条动画效果 */
.bg-gradient-to-r.from-yellow-400 {
  box-shadow: 0 0 10px rgba(251, 191, 36, 0.3);
}

.bg-gradient-to-r.from-gray-500 {
  box-shadow: 0 0 10px rgba(156, 163, 175, 0.2);
}
</style> 
