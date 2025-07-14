<script setup lang="ts">
import { gsap } from 'gsap'
import { onMounted, onUnmounted, ref } from 'vue'


const sectionRef = ref<HTMLElement>()
const dashboardRef = ref<HTMLElement>()
const progressRingRef = ref<HTMLElement>()
const statsCardsRef = ref<HTMLElement>()



const hasAnimated = ref(false)
const isAnimating = ref(false)
const currentStep = ref(0)


const dashboardData = ref({
  overallProgress: 0,
  bundleSize: {
    before: 0,
    after: 0,
    reduction: 0
  },
  filesProcessed: {
    total: 0,
    removed: 0,
    kept: 0
  },
  performance: {
    loadTime: 0,
    improvement: 0
  }
})


const statsCards = [
  {
    title: 'Bundle Size',
    beforeValue: '3.2MB',
    afterValue: '450KB',
    reduction: '86%',
    icon: '/Firth_box.svg',
    color: 'orange'
  },
  {
    title: 'Files Removed',
    beforeValue: '24',
    afterValue: '8',
    reduction: '16',
    icon: '/Firth_trash.svg',
    color: 'yellow'
  },
  {
    title: 'Load Time',
    beforeValue: '2.8s',
    afterValue: '0.3s',
    reduction: '89%',
    icon: '/Firth_speed.svg',
    color: 'blue'
  },
  {
    title: 'Dependencies',
    beforeValue: '156',
    afterValue: '23',
    reduction: '85%',
    icon: '/Firth_dependencies.svg',
    color: 'lightblue'
  }
]




let observer: IntersectionObserver | null = null
let mainTimeline: gsap.core.Timeline | null = null


const getOptimizedWidth = (card: any) => {
  switch (card.title) {
    case 'Bundle Size':
      return '14%'
    case 'Files Removed':
      return '33%'
    case 'Load Time':
      return '11%'
    case 'Dependencies':
      return '15%'
    default:
      return '0%'
  }
}


const runAnimation = async () => {
  if (!dashboardRef.value || !progressRingRef.value || !statsCardsRef.value) return
  if (isAnimating.value) return

  isAnimating.value = true


  mainTimeline = gsap.timeline({
    onComplete: () => {
      isAnimating.value = false
    }
  })


  const progressCircle = progressRingRef.value.querySelector('.progress-circle')
  const progressText = progressRingRef.value.querySelector('.progress-text')
  const cards = statsCardsRef.value.querySelectorAll('.stat-card')

  if (!progressCircle || !progressText) return


  gsap.set(dashboardRef.value, {
    opacity: 0,
    scale: 0.8
  })

  gsap.set(cards, {
    opacity: 0,
    y: 30,
    scale: 0.9
  })



  gsap.set(progressCircle, {
    strokeDasharray: "0 628"
  })


  mainTimeline

    .to(dashboardRef.value, {
      opacity: 1,
      scale: 1,
      duration: 0.8,
      ease: "back.out(1.7)",
      onComplete: () => {
        currentStep.value = 1
      }
    })


    .to(progressCircle, {
      strokeDasharray: "546 628",
      duration: 2.5,
      ease: "power2.out",
      onUpdate: function() {

        const progress = Math.round(this.progress() * 87)
        dashboardData.value.overallProgress = progress
        if (progressText) {
          progressText.textContent = `${progress}%`
        }
      }
    }, "+=0.3")


    .to(cards, {
      opacity: 1,
      y: 0,
      scale: 1,
      duration: 0.6,
      stagger: {
        amount: 1.2,
        ease: "power2.out"
      },
      ease: "back.out(1.3)",
      onComplete: () => {
        currentStep.value = 3
      }
    }, "-=1.5")




     .to(progressCircle, {
       filter: "drop-shadow(0 0 10px #fbbf24)",
       duration: 0.4,
       yoyo: true,
       repeat: 1
     }, "+=0.5")
}


const setupIntersectionObserver = () => {
  if (!sectionRef.value) return
  
  observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting && !hasAnimated.value && !isAnimating.value) {
  
          hasAnimated.value = true
          setTimeout(() => {
            runAnimation()
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
  
  isAnimating.value = false
})
</script>

<template>
  <div ref="sectionRef" class="min-h-screen flex flex-col justify-center items-center py-16">
    <div class="w-full max-w-7xl px-8">

      <div class="text-center mb-16">
        <h2 class="text-6xl font-bold mb-4 text-white">Optimization Dashboard</h2>
        <p class="text-2xl text-gray-400">Tree shaking & source maps included - Overall performance metrics</p>
      </div>

      
      <div ref="dashboardRef" class="relative opacity-0">
        
        <div class="flex justify-center mb-12">
          <div ref="progressRingRef" class="relative">
            <svg width="280" height="280" viewBox="0 0 280 280" class="transform -rotate-90">

              <circle
                cx="140"
                cy="140"
                r="100"
                stroke="rgb(31 41 55)"
                stroke-width="8"
                fill="none"
              />
              
              <circle
                cx="140"
                cy="140"
                r="100"
                stroke="url(#progressGradient)"
                stroke-width="8"
                fill="none"
                stroke-linecap="round"
                class="progress-circle"
                style="stroke-dasharray: 0 628; transition: stroke-dasharray 0.3s ease;"
              />

               <defs>
                 <linearGradient id="progressGradient" x1="0%" y1="0%" x2="100%" y2="0%">
                   <stop offset="0%" style="stop-color:#2563eb;stop-opacity:1" />
                   <stop offset="50%" style="stop-color:#fbbf24;stop-opacity:1" />
                   <stop offset="100%" style="stop-color:#f97316;stop-opacity:1" />
                 </linearGradient>
               </defs>
            </svg>
            
            <div class="absolute inset-0 flex flex-col items-center justify-center">
              <div class="progress-text text-5xl font-bold text-white mb-2">0%</div>
              <div class="text-lg text-gray-400">Overall Optimized</div>
            </div>
          </div>
        </div>

        
        <div ref="statsCardsRef" class="grid grid-cols-2 lg:grid-cols-4 gap-6 mb-12">
          <div 
            v-for="(card, index) in statsCards" 
            :key="index"
            class="stat-card backdrop-blur-xl bg-gradient-to-br from-gray-800/20 via-gray-700/10 to-gray-600/15 rounded-xl p-6 border border-white/10 shadow-2xl relative overflow-hidden opacity-0"
          >
            
            <div class="absolute inset-0 rounded-xl bg-gradient-to-br from-white/8 via-gray-400/4 to-white/6 pointer-events-none"></div>
            
            <div class="absolute top-0 left-0 right-0 h-px bg-gradient-to-r from-transparent via-white/15 to-transparent"></div>
            
            <div class="relative z-10">
              <div class="flex items-center gap-3 mb-4">
              <img :src="card.icon" class="w-8 h-8 filter drop-shadow-sm" alt="" />
              <h3 class="text-lg font-bold text-white">{{ card.title }}</h3>
            </div>
                         <div class="space-y-3">
               
               <div class="space-y-1">
                 <div class="flex justify-between items-center">
                   <span class="text-xs font-semibold text-gray-300 tracking-wide uppercase">Original</span>
                   <span class="text-sm font-bold text-gray-200">{{ card.beforeValue }}</span>
                 </div>
                 <div class="w-full bg-gray-800/60 rounded-full h-2 overflow-hidden backdrop-blur-sm border border-gray-600/30">
                   <div 
                     class="original-bar h-2 rounded-full transition-all duration-1500 ease-out" 
                     :style="{ width: currentStep >= 1 ? '100%' : '0%' }"
                   ></div>
                 </div>
               </div>
               
               
               <div class="space-y-1">
                 <div class="flex justify-between items-center">
                   <span class="text-xs font-semibold text-blue-300 tracking-wide uppercase">Optimized</span>
                   <span class="text-sm font-bold text-yellow-400 drop-shadow-sm">{{ card.afterValue }}</span>
                 </div>
                 <div class="w-full bg-gray-800/60 rounded-full h-2 overflow-hidden backdrop-blur-sm border border-gray-600/30">
                   <div 
                     class="optimized-bar h-2 rounded-full transition-all duration-2000 ease-out"
                     :style="{ width: currentStep >= 2 ? getOptimizedWidth(card) : '0%' }"
                   ></div>
                 </div>
               </div>
               
               
               <div class="text-center pt-4 border-t border-white/10 mt-4">
                 <div class="space-y-1">
                   <div class="text-xs text-gray-400 uppercase tracking-wide">IMPROVED</div>
                   <div class="text-4xl font-bold text-blue-400">{{ card.reduction }}</div>
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

.stat-card {
  transition: all 0.3s ease;
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
}

.stat-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.4);
  border-color: rgba(255, 255, 255, 0.2);
  background: linear-gradient(135deg, rgba(156, 163, 175, 0.15), rgba(107, 114, 128, 0.08), rgba(75, 85, 99, 0.12));
}




.progress-circle {
  filter: drop-shadow(0 0 8px rgba(251, 191, 36, 0.4));
}




.progress-text {
  font-variant-numeric: tabular-nums;
}


.optimized-bar {
  background: linear-gradient(90deg, #2563eb, #3b82f6, #fbbf24, #f97316);
  background-size: 200% 100%;
  box-shadow: 
    0 0 8px rgba(59, 130, 246, 0.4),
    inset 0 1px 0 rgba(255, 255, 255, 0.2);
  animation: shimmer 3s ease-in-out infinite;
  position: relative;
}

.optimized-bar::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.3),
    transparent
  );
  animation: sweep 2s ease-out;
  animation-delay: 0.5s;
}


@keyframes shimmer {
  0%, 100% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
}

@keyframes sweep {
  0% {
    left: -100%;
  }
  100% {
    left: 100%;
  }
}


.original-bar {
  background: linear-gradient(90deg, #6b7280, #9ca3af, #6b7280);
  background-size: 200% 100%;
  box-shadow: 
    0 0 4px rgba(107, 114, 128, 0.3),
    inset 0 1px 0 rgba(255, 255, 255, 0.1);
  animation: originalShimmer 4s ease-in-out infinite;
}

@keyframes originalShimmer {
  0%, 100% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
}


.bg-gray-800\/60 {
  box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.4);
}
</style> 
