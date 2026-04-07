<template>
  <div class="threshold-container">
    <div class="strategy-selector">
      <button v-for="s in strategies" :key="s.id" :class="{ active: currentStrategy === s.id }" @click="currentStrategy = s.id">
        {{ s.label }}
      </button>
    </div>

    <svg class="threshold-svg" viewBox="0 0 1000 450" role="img" aria-label="阈值策略原理演示">
      <defs>
        <filter id="glowEffect" x="-20%" y="-20%" width="140%" height="140%">
          <feGaussianBlur stdDeviation="3" result="blur" />
          <feComposite in="SourceGraphic" in2="blur" operator="over" />
        </filter>
        <linearGradient id="barGradient" x1="0" y1="0" x2="0" y2="1">
          <stop offset="0%" stop-color="#3b82f6" />
          <stop offset="100%" stop-color="#2563eb" />
        </linearGradient>
      </defs>

      <g class="header-text">
        <text x="150" y="40">含噪小波系数</text>
        <text x="500" y="40">阈值判断机制</text>
        <text x="850" y="40">去噪后的系数</text>
      </g>

      <g class="stage-left" transform="translate(50, 80)">
        <rect width="250" height="300" rx="10" fill="#f8fafc" stroke="#e2e8f0" stroke-width="2" />
        <g class="bars-input">
          <rect v-for="(h, i) in inputBars" :key="i" :x="20 + i * 18" :y="280 - h" width="10" :height="h" fill="#94a3b8" opacity="0.6" />
        </g>
        <text x="125" y="330" class="sub-label">输入信号 + 噪声</text>
      </g>

      <g class="stage-center" transform="translate(350, 80)">
        <rect width="300" height="300" rx="10" fill="#eff6ff" stroke="#3b82f6" stroke-width="2" stroke-dasharray="6 4" />

        <g class="coefficient-flow">
          <rect
            v-for="(h, i) in inputBars"
            :key="i"
            :x="30 + i * 20"
            :y="280 - h"
            width="12"
            :height="h"
            :fill="h > currentThreshold ? 'url(#barGradient)' : '#cbd5e1'"
            :class="{ 'is-signal': h > currentThreshold, 'is-noise': h <= currentThreshold }"
          />
        </g>

        <g class="threshold-line-group" :style="{ transform: `translateY(${280 - currentThreshold - 280}px)` }">
          <line x1="10" y1="280" x2="290" y2="280" stroke="#ef4444" stroke-width="3" stroke-dasharray="none" />
          <polygon points="290,280 305,273 305,287" fill="#ef4444" />
          <text x="310" y="285" fill="#ef4444" font-weight="bold">阈值 λ</text>
        </g>

        <text x="150" y="330" class="sub-label">{{ strategyInfo.desc }}</text>
      </g>

      <g class="stage-right" transform="translate(700, 80)">
        <rect width="250" height="300" rx="10" fill="#f0fdf4" stroke="#22c55e" stroke-width="2" />
        <g class="bars-output">
          <rect
            v-for="(h, i) in inputBars"
            :key="'out-' + currentStrategy + '-' + i"
            :x="20 + i * 18"
            :y="280 - (h > currentThreshold ? h : 0)"
            width="10"
            :height="h > currentThreshold ? h : 0"
            fill="#22c55e"
            :class="{ 'bar-active': h > currentThreshold, 'bar-hidden': h <= currentThreshold }"
          />
        </g>
        <text x="125" y="330" class="sub-label">纯净信号分量 (λ={{ currentThreshold }})</text>
      </g>

      <path class="connector" d="M 300 230 L 350 230" stroke="#cbd5e1" stroke-width="2" marker-end="url(#arrowHead)" />
      <path class="connector" d="M 650 230 L 700 230" stroke="#cbd5e1" stroke-width="2" marker-end="url(#arrowHead)" />

      <defs>
        <marker id="arrowHead" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto">
          <path d="M 0 0 L 10 5 L 0 10 z" fill="#cbd5e1" />
        </marker>
      </defs>
    </svg>

    <div class="strategy-details">
      <h3>{{ strategyInfo.title }}</h3>
      <p>{{ strategyInfo.detail }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const currentStrategy = ref('visu')

const strategies = [
  {
    id: 'visu',
    label: 'VisuShrink (固定)',
    threshold: 180,
    title: 'VisuShrink 通用阈值',
    desc: '全局固定阈值',
    detail: '采用统一的计算公式（λ = σ√(2lnN)），对所有子带使用相同的阈值。优点是简单稳健，但容易导致过度去噪，丢失细节。',
  },
  {
    id: 'sure',
    label: 'SureShrink (自适应)',
    threshold: 120,
    title: 'SureShrink 自适应阈值',
    desc: '基于Stein无偏似然估计',
    detail: '针对每个子带独立计算最优阈值。对于细节丰富的子带使用较小阈值，对于平滑子带使用较大阈值，能够更好地保留细节。',
  },
  {
    id: 'bayes',
    label: 'BayesShrink (统计优化)',
    threshold: 85,
    title: 'BayesShrink 统计阈值',
    desc: '广义高斯分布建模',
    detail: '在贝叶斯框架下，假设信号服从广义高斯分布。实验证明它在信噪比和视觉效果上表现最平衡，是目前主流的优化算法。',
  },
]

const inputBars = [120, 240, 60, 190, 80, 260, 110, 210, 50, 140, 230, 95]

const strategyInfo = computed(() => strategies.find(s => s.id === currentStrategy.value))
const currentThreshold = computed(() => strategyInfo.value.threshold)
</script>

<style scoped>
.threshold-container {
  background: white;
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.06);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.strategy-selector {
  display: flex;
  gap: 12px;
  background: #f1f5f9;
  padding: 6px;
  border-radius: 10px;
}

.strategy-selector button {
  padding: 8px 16px;
  border: none;
  background: transparent;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
  color: #64748b;
  transition: all 0.3s;
}

.strategy-selector button.active {
  background: white;
  color: #2563eb;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.threshold-svg {
  width: 100%;
  height: auto;
  max-width: 900px;
}

.header-text text {
  font-size: 18px;
  font-weight: 700;
  fill: #1e293b;
  text-anchor: middle;
}

.sub-label {
  font-size: 14px;
  fill: #64748b;
  font-weight: 500;
  text-anchor: middle;
}

.threshold-line-group {
  transition: transform 0.6s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.is-noise {
  opacity: 0.3;
  transition: all 0.5s;
}

.is-signal {
  opacity: 1;
  filter: drop-shadow(0 0 2px rgba(37, 99, 235, 0.3));
  transition: all 0.5s;
}

.bars-output rect {
  transition: all 0.6s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.bar-active {
  opacity: 1;
}

.bar-hidden {
  opacity: 0;
  transform: scaleY(0);
  transform-origin: bottom;
}

.strategy-details {
  width: 100%;
  max-width: 800px;
  text-align: left;
  border-top: 1px dashed #e2e8f0;
  padding-top: 15px;
}

.strategy-details h3 {
  color: #2563eb;
  margin-bottom: 8px;
  font-size: 18px;
}

.strategy-details p {
  font-size: 14px;
  line-height: 1.6;
  color: #475569;
}

@keyframes signal-jitter {
  0%,
  100% {
    transform: scaleY(1);
  }
  50% {
    transform: scaleY(1.02);
  }
}

.is-signal {
  animation: signal-jitter 2s infinite ease-in-out;
  transform-origin: bottom;
}
</style>
