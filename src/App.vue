<template>
  <div id="app">
    <transition name="fade" v-if="showIntro">
      <div class="intro-container" @click="showIntro = false">
        <!-- 背景视频 (冷色滤镜) -->
        <video
          class="video-background"
          :src="videoSrc"
          autoplay
          muted
          loop
          playsinline
        ></video>

        <!-- 机械装饰层 -->
        <div class="grid-bg"></div>
        <div class="hex-bg"></div>
  
        <div class="data-flow"></div>
        <div class="red-dot red-dot-1"></div>
        <div class="red-dot red-dot-2"></div>
        <div class="red-dot red-dot-3"></div>
        <div class="red-dot red-dot-4"></div>
        <div class="mech-corner"></div>

        <!-- 主内容 -->
        <div class="intro-content" aria-live="polite">
          <div class="intro-inner">
            <!-- 打字机区域 -->
            <div class="typewriter-wrap">
              <p class="typewriter">
                {{ displayText }}
              </p>
              <span class="cursor"></span>
            </div>
          </div>
        </div>
      </div>
    </transition>
    <div v-else>
      <navbar />
      <main class="main-content">
        <RouterView />
      </main>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";
import { RouterView } from "vue-router";
import navbar from "@/components/navbar.vue";

const showIntro = ref(true);
const videoSrc = ref("");

// 莫宁语录 (星枢演构)
const lines = [
  "白发红瞳，星枢演构",
  "跨越时空的鸿沟，以双手触碰世界的宏大",
  "深空联合研究院 · 星炬学院隧者工学部教授",
  "用科技延伸自身，以计算推演未来",
  "机械义肢之下，藏着温柔的梦想",
  "星枢演构 · 群星点亮之时",
  "冷静的外表，炽热的内心",
  "正因为能看到星星，人类才会对天空之外的世界产生好奇",
  "星枢演构，启动",
  "以机械之躯，触碰宇宙的真理",
] as const;

const displayText = ref("");
let typingTimer: number | null = null;
const typingSpeed = 120; // 略快于莫宁原版

function pickRandomLine(): string {
  const idx = Math.floor(Math.random() * lines.length);
  return lines[idx];
}

function startOnceType(line: string) {
  // 检测用户是否偏好减少动效
  const reduce =
    window.matchMedia &&
    window.matchMedia("(prefers-reduced-motion: reduce)").matches;
  if (reduce) {
    displayText.value = line;
    return;
  }

  let i = 0;
  typingTimer = window.setInterval(() => {
    i++;
    displayText.value = line.slice(0, i);
    if (i >= line.length) {
      if (typingTimer) {
        clearInterval(typingTimer);
        typingTimer = null;
      }
    }
  }, typingSpeed);
}

onMounted(() => {
  // 随机选取视频 (沿用原有逻辑，但可调整目录)
  const isMobile = window.innerWidth <= 768;
  const folder = isMobile ? "/mp2" : "/mp1";
  const index = Math.floor(Math.random() * 4) + 1;
  videoSrc.value = `${folder}/1 (${index}).mp4`;

  // 6秒后自动关闭 (与原有一致)
  setTimeout(() => {
    showIntro.value = false;
  }, 6000);

  const line = pickRandomLine();
  setTimeout(() => startOnceType(line), 200);
});

onBeforeUnmount(() => {
  if (typingTimer) clearInterval(typingTimer);
});
</script>

<style scoped lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;800&family=Rajdhani:wght@300;500;600&family=Share+Tech+Mono&display=swap");

#app {
  position: relative;
  min-height: 100vh;
  animation: cursorAnimation 1s infinite step-start;
}

/* 淡入淡出动画 (保持不变) */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 1s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* ===== 莫宁风格开屏容器 ===== */
.intro-container {
  position: fixed;
  inset: 0;
  background: radial-gradient(ellipse at center, #0a0c16 0%, #03050f 100%);
  z-index: 9999;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
  flex-direction: column;
}

/* 背景视频 (冷色滤镜) */
.video-background {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0.5; // 降低原视频不透明度，让机械层更突出
 
  z-index: 1;
  pointer-events: none;
}

/* ===== 机械装饰层 ===== */
.grid-bg {
  position: absolute;
  inset: 0;
  background-image: linear-gradient(
      rgba(60, 140, 230, 0.04) 1px,
      transparent 1px
    ),
    linear-gradient(90deg, rgba(60, 140, 230, 0.04) 1px, transparent 1px);
  background-size: 40px 40px;
  z-index: 2;
  pointer-events: none;
}

.hex-bg {
  position: absolute;
  inset: 0;
  background-image: repeating-linear-gradient(
    45deg,
    rgba(74, 165, 255, 0.03) 0px,
    rgba(74, 165, 255, 0.03) 2px,
    transparent 2px,
    transparent 30px
  );
  z-index: 2;
  pointer-events: none;
}



/* 数据流 */
.data-flow {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 1px;
  background: linear-gradient(
    90deg,
    transparent,
    #4aa5ff 20%,
    #9ac9ff 50%,
    #4aa5ff 80%,
    transparent
  );
  filter: blur(2px);
  animation: flowMove 4s linear infinite;
  z-index: 3;
  opacity: 0.6;
}

@keyframes flowMove {
  0% {
    transform: translateX(-100%);
  }
  100% {
    transform: translateX(100%);
  }
}

/* 红瞳微光 (极小点缀) */
.red-dot {
  position: absolute;
  width: 4px;
  height: 4px;
  background: #ff6b8b;
  border-radius: 50%;
  filter: blur(2px);
  box-shadow: 0 0 12px #ff6b8b;
  opacity: 0.5;
  animation: redBreath 3s infinite;
  z-index: 3;
  pointer-events: none;
}
.red-dot-1 {
  top: 15%;
  left: 10%;
}
.red-dot-2 {
  top: 25%;
  right: 15%;
}
.red-dot-3 {
  bottom: 20%;
  left: 20%;
}
.red-dot-4 {
  bottom: 30%;
  right: 25%;
}

@keyframes redBreath {
  0%,
  100% {
    opacity: 0.2;
    transform: scale(1);
  }
  50% {
    opacity: 0.8;
    transform: scale(1.5);
  }
}

/* 左上机械角 */
.mech-corner {
  position: absolute;
  top: 20px;
  left: 20px;
  width: 60px;
  height: 60px;
  background: radial-gradient(
    circle at 30% 30%,
    rgba(74, 165, 255, 0.15),
    transparent 70%
  );
  clip-path: polygon(
    25% 0%,
    75% 0%,
    100% 25%,
    100% 75%,
    75% 100%,
    25% 100%,
    0% 75%,
    0% 25%
  );
  animation: rotateSlow 20s linear infinite;
  z-index: 2;
  opacity: 0.3;
}

@keyframes rotateSlow {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

/* ===== 主内容 ===== */
.intro-content {
  width: 100%;
  box-sizing: border-box;
  padding: 18px 12px;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10;

  .intro-inner {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 20px;
    transition: opacity 420ms ease, transform 420ms ease;
    opacity: 1;
    transform: translateY(0);

    .typewriter-wrap {
      display: flex;
      align-items: center;
      gap: 12px;
      background: rgba(10, 15, 30, 0.5);
      backdrop-filter: blur(10px);
      padding: 20px 40px;
      border-radius: 60px;
      border: 1px solid rgba(74, 165, 255, 0.4);
      box-shadow: 0 0 40px rgba(30, 100, 200, 0.3),
        inset 0 0 20px rgba(74, 165, 255, 0.2);

      .typewriter {
        margin: 0;
        font-family: "Orbitron", "Rajdhani", sans-serif;
        font-weight: 600;
        font-size: clamp(28px, 6vw, 48px);
        line-height: 1.2;
        background: linear-gradient(135deg, #ffffff, #b0e0ff, #5a9eff);
        -webkit-background-clip: text;
        background-clip: text;
        color: transparent;
        text-shadow: 0 0 20px rgba(90, 158, 255, 0.5),
          2px 2px 4px rgba(0, 0, 0, 0.5);
        -webkit-text-stroke: 1px rgba(0, 20, 40, 0.3);
        letter-spacing: 2px;
      }

      .cursor {
        width: 8px;
        height: 2.4em;
        background: #4aa5ff;
        border-radius: 2px;
        animation: blink 1s step-end infinite;
        box-shadow: 0 0 15px #4aa5ff;
      }
    }
  }
}

@keyframes blink {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0;
  }
}

/* 移动端适配 */
@media (max-width: 768px) {
  .intro-inner {
    flex-direction: column;
    gap: 8px;
    text-align: center;
  }

  .typewriter-wrap {
    padding: 15px 25px;
  }

  .typewriter {
    font-size: clamp(20px, 5vw, 32px);
  }

  .mech-corner {
    width: 40px;
    height: 40px;
  }
}
</style>
