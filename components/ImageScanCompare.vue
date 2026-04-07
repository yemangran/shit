<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from "vue";

const props = withDefaults(
    defineProps<{
        noisySrc: string;
        cleanSrc: string;
        noisyLabel?: string;
        cleanLabel?: string;
        durationMs?: number;
    }>(),
    {
        noisyLabel: "含噪图像",
        cleanLabel: "去噪图像",
        durationMs: 3600,
    },
);

const progress = ref(0.5);
let rafId = 0;
let startTime = 0;

const scanLeft = computed(() => `${progress.value * 100}%`);
const cleanClip = computed(() => `inset(0 0 0 ${progress.value * 100}%)`);

function animate(timestamp: number) {
    if (!startTime) startTime = timestamp;

    const elapsed = timestamp - startTime;
    const cycle = (elapsed % props.durationMs) / props.durationMs;

    // 0..1..0 往返运动
    progress.value = 0.5 - 0.5 * Math.cos(cycle * Math.PI * 2);
    rafId = requestAnimationFrame(animate);
}

onMounted(() => {
    rafId = requestAnimationFrame(animate);
});

onBeforeUnmount(() => {
    cancelAnimationFrame(rafId);
});
</script>

<template>
    <div class="compare-card">
        <div class="compare-stage">
            <img class="base-image" :src="noisySrc" :alt="noisyLabel" />
            <img
                class="clean-image"
                :src="cleanSrc"
                :alt="cleanLabel"
                :style="{ clipPath: cleanClip }"
            />

            <div class="scan-line" :style="{ left: scanLeft }" />

            <div class="side-label noisy">{{ noisyLabel }}</div>
            <div class="side-label clean">{{ cleanLabel }}</div>
        </div>
    </div>
</template>

<style scoped>
.compare-card {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
}

.compare-stage {
  position: relative;
  width: min(100%, 760px);
  aspect-ratio: 40 / 39;
  overflow: hidden;
  border-radius: 18px;
  background: #0f172a;
    box-shadow: 0 18px 40px rgba(15, 23, 42, 0.18);
}

.base-image,
.clean-image {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
}

.scan-line {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 3px;
    background: linear-gradient(
        180deg,
        rgba(255, 255, 255, 0.1),
        rgba(255, 255, 255, 0.95),
        rgba(255, 255, 255, 0.1)
    );
    box-shadow:
        0 0 0 1px rgba(255, 255, 255, 0.35),
        0 0 18px rgba(255, 255, 255, 0.65),
        0 0 36px rgba(59, 130, 246, 0.45);
    transform: translateX(-50%);
}

.scan-line::before {
    content: "";
    position: absolute;
    inset: 0;
    width: 18px;
    left: 50%;
    transform: translateX(-50%);
    background: linear-gradient(
        90deg,
        rgba(255, 255, 255, 0),
        rgba(255, 255, 255, 0.25),
        rgba(255, 255, 255, 0)
    );
}

.side-label {
    position: absolute;
    bottom: 14px;
    padding: 4px 10px;
    border-radius: 999px;
    font-size: 12px;
    color: white;
    background: rgba(15, 23, 42, 0.58);
    backdrop-filter: blur(6px);
}

.side-label.noisy {
    left: 14px;
}

.side-label.clean {
    right: 14px;
}
</style>
