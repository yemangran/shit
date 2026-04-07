<template>
  <div class="wavelet-container">
    <svg class="wavelet-svg" viewBox="0 0 1000 500" role="img" aria-label="小波分解原理演示">
      <!-- 滤镜定义 -->
      <defs>
        <filter id="softGlow" x="-20%" y="-20%" width="140%" height="140%">
          <feGaussianBlur stdDeviation="2" result="blur" />
          <feComposite in="SourceGraphic" in2="blur" operator="over" />
        </filter>
        
        <!-- 小波基形状：Haar小波示意（以0,0为中心重新定义） -->
        <path id="waveletBase" d="M -30 10 L -10 10 L -10 -10 L 10 -10 L 10 10 L 30 10" fill="none" stroke="#2563eb" stroke-width="2" />
      </defs>

      <!-- 标题部分 -->
      <g class="header-text">
        <text x="150" y="40">1. 滑窗扫描 (窗口与图像卷积)</text>
        <text x="500" y="40">2. 特征映射</text>
        <text x="820" y="40">3. 象限分解结果</text>
      </g>

      <!-- 左侧：原始图像与扫描窗口 -->
      <g class="stage-left" transform="translate(50, 80)">
        <!-- 背景框 -->
        <rect width="300" height="300" rx="12" fill="#f8fafc" stroke="#e2e8f0" stroke-width="2" />
        
        <!-- 模拟图像内容：简单几何体 -->
        <g class="image-content">
          <circle cx="150" cy="120" r="50" fill="#94a3b8" opacity="0.3" />
          <rect x="100" y="180" width="100" height="60" fill="#64748b" opacity="0.4" />
          <!-- 模拟高频噪点 -->
          <circle v-for="n in 20" :key="n" :cx="Math.random()*200+50" :cy="Math.random()*200+50" r="1.5" fill="#cbd5e1" />
        </g>

        <!-- 扫描窗口 (Wavelet Window) -->
        <g class="scan-window">
          <rect x="0" y="0" width="60" height="60" fill="none" stroke="#2563eb" stroke-width="3" />
          <path d="M 10 30 L 25 30 L 25 15 L 40 15 L 40 30 L 55 30" fill="none" stroke="#2563eb" stroke-width="1.5" />
          <rect x="0" y="0" width="60" height="60" fill="rgba(37, 99, 235, 0.1)" />
        </g>
        
        <text x="150" y="330" class="sub-label">原始图像 f(x,y)</text>
      </g>

      <!-- 中间：小波基「转化器」 -->
      <g class="stage-center" transform="translate(420, 150)">
        <!-- 核心透镜 -->
        <circle cx="80" cy="80" r="70" fill="#eff6ff" stroke="#3b82f6" stroke-width="3" stroke-dasharray="8 4" />
        <use href="#waveletBase" transform="translate(80, 80) scale(1.4)" />
        <text x="80" y="175" class="sub-label">小波基函数 Ψ</text>
        
        <!-- 动态投影线 -->
        <path class="projection-line" d="M -70 60 L 0 80" stroke="#94a3b8" stroke-width="1" />
        <path class="projection-line" d="M 150 80 L 280 0" stroke="#94a3b8" stroke-width="1" />
        <path class="projection-line" d="M 150 80 L 280 230" stroke="#94a3b8" stroke-width="1" />
      </g>

      <!-- 右侧：四象限物理拆解 -->
      <g class="stage-right" transform="translate(680, 80)">
        <!-- LL: 低频轮廓 (缩小的原图) -->
        <g transform="translate(0, 0)">
          <g class="quadrant ll">
            <rect width="140" height="140" rx="10" fill="#f0fdf4" stroke="#22c55e" stroke-width="2" />
            <circle cx="70" cy="60" r="30" fill="#22c55e" opacity="0.6" />
            <rect x="45" y="95" width="50" height="25" fill="#22c55e" opacity="0.6" />
            <text x="70" y="165" class="quad-label">LL (近似/轮廓)</text>
            <text x="70" y="75" class="quad-tag">低频</text>
          </g>
        </g>

        <!-- LH: 水平细节 -->
        <g transform="translate(160, 0)">
          <g class="quadrant lh">
            <rect width="140" height="140" rx="10" fill="#fff7ed" stroke="#f97316" stroke-width="2" />
            <line x1="20" y1="45" x2="120" y2="45" stroke="#f97316" stroke-width="3" opacity="0.8" />
            <line x1="20" y1="95" x2="120" y2="95" stroke="#f97316" stroke-width="3" opacity="0.8" />
            <text x="70" y="165" class="quad-label">LH (水平细节)</text>
          </g>
        </g>

        <!-- HL: 垂直细节 -->
        <g transform="translate(0, 185)">
          <g class="quadrant hl">
            <rect width="140" height="140" rx="10" fill="#fef2f2" stroke="#ef4444" stroke-width="2" />
            <line x1="45" y1="20" x2="45" y2="120" stroke="#ef4444" stroke-width="3" opacity="0.8" />
            <line x1="95" y1="20" x2="95" y2="120" stroke="#ef4444" stroke-width="3" opacity="0.8" />
            <text x="70" y="165" class="quad-label">HL (垂直细节)</text>
          </g>
        </g>

        <!-- HH: 对角线细节 -->
        <g transform="translate(160, 185)">
          <g class="quadrant hh">
            <rect width="140" height="140" rx="10" fill="#f5f3ff" stroke="#8b5cf6" stroke-width="2" />
            <circle v-for="n in 15" :key="n" :cx="Math.random()*110+15" :cy="Math.random()*110+15" r="1.5" fill="#8b5cf6" />
            <text x="70" y="165" class="quad-label">HH (对角线/噪声)</text>
          </g>
        </g>
      </g>

      <!-- 总结文案 -->
      <g class="summary-text" transform="translate(500, 470)">
        <text text-anchor="middle" fill="#475569">
          小波基如同一把<tspan fill="#2563eb" font-weight="bold">数学筛子</tspan>，通过平移和伸缩，将图像中的不同频率特征提取到不同象限。
        </text>
      </g>
    </svg>
  </div>
</template>

<style scoped>
.wavelet-container {
  background: white;
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.05);
}

.wavelet-svg {
  width: 100%;
  height: auto;
  font-family: 'Inter', system-ui, sans-serif;
}

.header-text text {
  font-size: 18px;
  font-weight: 700;
  fill: #1e293b;
  text-anchor: middle;
}

.sub-label {
  font-size: 16px;
  fill: #64748b;
  font-weight: 500;
}

.quad-label {
  font-size: 14px;
  fill: #334155;
  font-weight: 600;
  text-anchor: middle;
}

.quad-tag {
  font-size: 24px;
  font-weight: 800;
  fill: #22c55e;
  opacity: 0.15;
  text-anchor: middle;
}

/* 1. 扫描窗口动画：Z字型扫描 */
.scan-window {
  animation: zigzag-scan 6s linear infinite;
}

@keyframes zigzag-scan {
  0%   { transform: translate(0, 0); }
  20%  { transform: translate(240px, 0); }
  25%  { transform: translate(0, 80px); }
  45%  { transform: translate(240px, 80px); }
  50%  { transform: translate(0, 160px); }
  70%  { transform: translate(240px, 160px); }
  75%  { transform: translate(0, 240px); }
  95%  { transform: translate(240px, 240px); }
  100% { transform: translate(0, 0); }
}

/* 2. 滤波器脉冲效果 */
.stage-center circle {
  animation: pulse-lens 2s ease-in-out infinite;
}

@keyframes pulse-lens {
  0%, 100% { stroke-width: 3; filter: none; }
  50% { stroke-width: 5; filter: url(#softGlow); transform: scale(1.02); transform-origin: center; }
}

/* 3. 投影线动画 */
.projection-line {
  stroke-dasharray: 5 5;
  animation: line-flow 1s linear infinite;
}

@keyframes line-flow {
  from { stroke-dashoffset: 10; }
  to { stroke-dashoffset: 0; }
}

/* 4. 四象限显现动画：模拟「提取」过程 */
.quadrant {
  animation: quadrant-pulse 6s ease-in-out infinite;
}

.ll { animation-delay: 0s; }
.lh { animation-delay: 0.5s; }
.hl { animation-delay: 1s; }
.hh { animation-delay: 1.5s; }

@keyframes quadrant-pulse {
  0%, 40% { opacity: 0.3; transform: scale(0.95); }
  50% { opacity: 1; transform: scale(1.05); filter: url(#softGlow); }
  60%, 100% { opacity: 1; transform: scale(1); }
}

/* 内容元素的微动 */
.ll circle, .ll rect { animation: float 3s ease-in-out infinite; }
@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-3px); }
}
</style>
