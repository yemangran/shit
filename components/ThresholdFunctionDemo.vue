<template>
  <div class="func-container">
    <!-- 顶部选择器 -->
    <div class="func-selector">
      <button v-for="f in functions" :key="f.id" :class="{ active: currentFunc === f.id }" @click="currentFunc = f.id">
        {{ f.label }}
      </button>
    </div>

    <!-- 核心动画区域 -->
    <div class="main-layout">
      <svg class="func-svg" viewBox="0 0 500 280" role="img">
        <defs>
          <!-- 坐标轴箭头 -->
          <marker id="axisArrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5" markerHeight="5" orient="auto">
            <path d="M 0 0 L 10 5 L 0 10 z" fill="#94a3b8" />
          </marker>
          <!-- 曲线发光滤镜 -->
          <filter id="lineGlow" x="-20%" y="-20%" width="140%" height="140%">
            <feGaussianBlur stdDeviation="2" result="blur" />
            <feComposite in="SourceGraphic" in2="blur" operator="over" />
          </filter>
        </defs>

        <!-- 背景微网格 -->
        <g class="grid" opacity="0.1">
          <line v-for="n in 9" :key="'v'+n" :x1="n*50" y1="20" :x2="n*50" y2="250" stroke="#64748b" stroke-width="0.5" />
          <line v-for="n in 5" :key="'h'+n" x1="50" :y1="n*50" x2="450" :y2="n*50" stroke="#64748b" stroke-width="0.5" />
        </g>

        <!-- 坐标轴 -->
        <g class="axes">
          <line x1="50" y1="135" x2="460" y2="135" stroke="#94a3b8" stroke-width="1.5" marker-end="url(#axisArrow)" />
          <line x1="250" y1="255" x2="250" y2="25" stroke="#94a3b8" stroke-width="1.5" marker-end="url(#axisArrow)" />
          <text x="465" y="152" class="axis-label">x</text>
          <text x="232" y="18" class="axis-label">f(x)</text>
        </g>

        <!-- 阈值边界引导线 -->
        <g class="threshold-guides" opacity="0.4">
          <line :x1="250 + lambdaPx" y1="40" :x2="250 + lambdaPx" y2="230" stroke="#ef4444" stroke-width="1" stroke-dasharray="3 3" />
          <line :x1="250 - lambdaPx" y1="40" :x2="250 - lambdaPx" y2="230" stroke="#ef4444" stroke-width="1" stroke-dasharray="3 3" />
        </g>

        <!-- 恒等辅助线 (y=x) -->
        <line x1="130" y1="255" x2="370" y2="15" stroke="#cbd5e1" stroke-width="1" stroke-dasharray="4 4" />

        <!-- 动态函数曲线 -->
        <path :d="functionPath" fill="none" stroke="#3b82f6" stroke-width="3.5" stroke-linecap="round" class="main-path" filter="url(#lineGlow)" />

        <!-- 实时追踪点 -->
        <g class="tracer-group">
          <circle :cx="250 + tracerX" :cy="135 - tracerY" r="8" fill="#f59e0b" opacity="0.3">
            <animate attributeName="r" values="7;10;7" dur="1.5s" repeatCount="indefinite" />
          </circle>
          <circle :cx="250 + tracerX" :cy="135 - tracerY" r="4.5" fill="#f59e0b" />
        </g>

        <!-- 阈值标记 λ (极致精简) -->
        <g class="lambda-labels">
          <text :x="250 + lambdaPx" y="145" text-anchor="middle" fill="#fca5a5" font-size="6" font-weight="600">λ</text>
          <text :x="250 - lambdaPx" y="145" text-anchor="middle" fill="#fca5a5" font-size="6" font-weight="600">-λ</text>
        </g>
      </svg>
    </div>

    <!-- 底部详情区域 -->
    <div class="func-details">
      <div class="detail-header">
        <h3 class="detail-title">{{ activeFunc.title }}</h3>
        <span class="visual-tag" :class="currentFunc">{{ activeFunc.hint }}</span>
      </div>
      
      <div class="formula-box">
        <span class="formula-label">数学表达式：</span>
        <code class="formula-code">{{ activeFunc.formula }}</code>
      </div>
      
      <div class="points-container">
        <div v-for="point in activeFunc.points" :key="point" class="point-card">
          <span class="dot"></span>{{ point }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const currentFunc = ref('hard')
const lambda = 75 
const lambdaPx = lambda

const functions = [
  { 
    id: 'hard', 
    label: '硬阈值', 
    title: '硬阈值函数 (Hard Thresholding)',
    formula: 'f(x) = x (|x| > λ) ; 0 (|x| ≤ λ)',
    hint: '不连续性',
    points: ['保留强信号系数', '噪声直接置零', '边缘锐利保真', '在 ±λ 处跳变']
  },
  { 
    id: 'soft', 
    label: '软阈值', 
    title: '软阈值函数 (Soft Thresholding)',
    formula: 'f(x) = sgn(x)(|x|-λ) (|x| > λ) ; 0',
    hint: '连续平滑',
    points: ['系数向零收缩', '整体数学连续', '有效抑制震荡', '易致边缘模糊']
  },
  { 
    id: 'improved', 
    label: '改进阈值', 
    title: '改进半软阈值 (Improved)',
    formula: 'f(x) = 自适应优化曲线',
    hint: '性能最优',
    points: ['消除硬阈值跳变', '减小软阈值偏差', '细节保持力强', '综合评价最高']
  }
]

const activeFunc = computed(() => functions.find(f => f.id === currentFunc.value))

const functionPath = computed(() => {
  const L = lambda
  const startX = -120
  const endX = 120
  
  const getPointY = (x) => {
    if (currentFunc.value === 'hard') {
      return Math.abs(x) > L ? x : 0
    } else if (currentFunc.value === 'soft') {
      return Math.abs(x) > L ? (x > 0 ? x - L : x + L) : 0
    } else {
      // 改进阈值：基于 Garrote 函数的平滑变形，确保在 (Hard, Soft) 之间
      return Math.abs(x) > L ? (x > 0 ? x - (L * L) / x * 0.5 : x - (L * L) / x * 0.5) : 0
    }
  }

  if (currentFunc.value === 'hard') {
    return `M ${250 + startX} ${135 - startX} L ${250 - L} ${135 + L} M ${250 - L} 135 L ${250 + L} 135 M ${250 + L} ${135 - L} L ${250 + endX} ${135 - endX}`
  }
  
  let path = `M ${250 + startX} ${135 - getPointY(startX)}`
  for (let x = startX + 2; x <= endX; x += 2) {
    path += ` L ${250 + x} ${135 - getPointY(x)}`
  }
  return path
})

const tracerPos = ref(0)
const tracerX = computed(() => tracerPos.value)
const tracerY = computed(() => {
  const x = tracerPos.value
  const L = lambda
  if (currentFunc.value === 'hard') return Math.abs(x) > L ? x : 0
  if (currentFunc.value === 'soft') return Math.abs(x) > L ? (x > 0 ? x - L : x + L) : 0
  return Math.abs(x) > L ? (x > 0 ? x - (L * L) / x * 0.5 : x - (L * L) / x * 0.5) : 0
})

onMounted(() => {
  let speed = 1.3
  const animate = () => {
    tracerPos.value += speed
    if (tracerPos.value > 120 || tracerPos.value < -120) speed *= -1
    requestAnimationFrame(animate)
  }
  animate()
})
</script>

<style scoped>
.func-container {
  background: #ffffff;
  border-radius: 16px;
  padding: 16px;
  box-shadow: 0 10px 25px rgba(0,0,0,0.05);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  border: 1px solid #f1f5f9;
}

.func-selector {
  display: flex;
  gap: 8px;
  background: #f8fafc;
  padding: 4px;
  border-radius: 12px;
  border: 1px solid #f1f5f9;
}

.func-selector button {
  padding: 6px 16px;
  border: none;
  background: transparent;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  font-size: 13px;
  color: #64748b;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.func-selector button.active {
  background: #ffffff;
  color: #2563eb;
  box-shadow: 0 4px 10px rgba(37,99,235,0.12);
}

.main-layout {
  width: 100%;
  display: flex;
  justify-content: center;
}

.func-svg {
  width: 100%;
  max-width: 420px;
  height: auto;
  display: block;
}

.axis-label {
  font-size: 14px;
  fill: #64748b;
  font-weight: 600;
  font-style: italic;
}

.func-details {
  width: 100%;
  border-top: 1px solid #f1f5f9;
  padding-top: 12px;
}

.detail-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.detail-title {
  font-size: 16px;
  color: #1e293b;
  margin: 0;
  font-weight: 700;
}

.visual-tag {
  padding: 2px 10px;
  border-radius: 20px;
  font-size: 11px;
  font-weight: bold;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.visual-tag.hard { background: #fee2e2; color: #ef4444; }
.visual-hint.soft { background: #dcfce7; color: #16a34a; }
.visual-tag.improved { background: #dbeafe; color: #2563eb; }

.formula-box {
  background: #f8fafc;
  padding: 8px 12px;
  border-radius: 8px;
  margin-bottom: 10px;
  display: flex;
  align-items: center;
  border: 1px solid #f1f5f9;
}

.formula-label {
  font-size: 12px;
  color: #64748b;
  font-weight: 500;
}

.formula-code {
  font-family: 'Fira Code', 'Courier New', monospace;
  font-size: 12px;
  color: #2563eb;
  font-weight: 600;
}

.points-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 6px 12px;
}

.point-card {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 12px;
  color: #475569;
  line-height: 1.2;
}

.dot {
  width: 5px;
  height: 5px;
  border-radius: 50%;
  background: #3b82f6;
  flex-shrink: 0;
}

.main-path {
  transition: d 0.4s ease-in-out;
}
</style>
