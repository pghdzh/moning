<template>
  <div class="morning-home">
    <!-- 背景轮播（两组） -->
    <div class="carousel carousel1" aria-hidden="true">
      <img
        v-for="(src, idx) in randomFive"
        :key="'c1-' + idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>
    <div class="carousel carousel2" aria-hidden="true">
      <img
        v-for="(src, idx) in randomFive2"
        :key="'c2-' + idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>

    <!-- 背景网格 -->
    <div class="grid-bg"></div>
    <div class="hex-bg"></div>

    <!-- 粒子画布 (动态数据演算) -->
    <canvas ref="particleCanvas" class="particle-canvas"></canvas>

    <!-- 全息演算圆盘 -->
    <div class="hologram-disk"></div>

    <!-- 红瞳微光 (极小点缀，四个角落) -->
    <div
      v-for="(pos, index) in redDotPositions"
      :key="'red-' + index"
      class="red-dot"
      :style="{
        top: pos.top,
        left: pos.left,
        right: pos.right,
        bottom: pos.bottom,
      }"
    ></div>

    <!-- 左上机械角 -->
    <div class="mech-corner"></div>

    <div class="data-flow"></div>

    <!-- 新增装饰：环绕标题的光环、浮动粒子、旋转环 -->
    <div class="title-ring"></div>
    <div class="floating-particles">
      <span
        v-for="n in 8"
        :key="'part' + n"
        class="particle"
        :style="getParticleStyle(n)"
      ></span>
    </div>
    <div class="rotating-ring"></div>

    <!-- 主内容区 (增强标题可见性) -->
    <div class="content">
      <div class="title-backdrop"></div>
      <h1 class="main-title">
        <span class="title-brackets">莫宁 · 星枢演构</span>
      </h1>
      <div class="subtitle-area">
        <span class="typed-text">{{ typedText }}</span>
        <span class="cursor"></span>
      </div>
    </div>

    <!-- 底部信息栏 -->
    <div class="footer-info">
      <span
        >✦ 深空联合研究院 · 星炬学院 · 隧者工学部 ✦ 白发红瞳 · 机械义肢 ·
        星枢演构 ✦</span
      >
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";

// 图片轮播数据
const modules = import.meta.glob("@/assets/images1/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const allSrcs = Object.values(modules).map((mod) => mod.default);
const modules2 = import.meta.glob("@/assets/images2/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const allSrcs2 = Object.values(modules2).map((mod) => mod.default);

function shuffle(arr) {
  const a = arr.slice();
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}
const randomFive = ref(shuffle(allSrcs).slice(0, 5));
const randomFive2 = ref(shuffle(allSrcs2).slice(0, 5));
const currentIndex = ref(0);
let imgTimer;

// 打字机文本
const typedText = ref("");

// 红点位置
const redDotPositions = [
  { top: "15%", left: "10%", right: "auto", bottom: "auto" },
  { top: "25%", right: "15%", left: "auto", bottom: "auto" },
  { bottom: "20%", left: "20%", top: "auto", right: "auto" },
  { bottom: "30%", right: "25%", top: "auto", left: "auto" },
];

// 浮动粒子样式
const getParticleStyle = (n) => {
  const left = Math.random() * 100 + "%";
  const top = Math.random() * 100 + "%";
  const delay = Math.random() * 5 + "s";
  const duration = Math.random() * 6 + 4 + "s";
  return { left, top, animationDelay: delay, animationDuration: duration };
};

// ---------- 粒子系统 ----------
const particleCanvas = ref(null);
let ctx = null,
  width = 0,
  height = 0;
let particles = [];
let animationFrame = null;
const PARTICLE_COUNT = 70;
const CONNECT_DIST = 140;

const initParticles = () => {
  particles = [];
  for (let i = 0; i < PARTICLE_COUNT; i++) {
    particles.push({
      x: Math.random() * width,
      y: Math.random() * height,
      vx: (Math.random() - 0.5) * 0.2,
      vy: (Math.random() - 0.5) * 0.1,
      size: Math.random() * 2 + 1,
      opacity: Math.random() * 0.4 + 0.2,
    });
  }
};

const resizeCanvas = () => {
  if (!particleCanvas.value) return;
  const canvas = particleCanvas.value;
  width = window.innerWidth;
  height = window.innerHeight;
  canvas.width = width;
  canvas.height = height;
  ctx = canvas.getContext("2d");
  initParticles();
};

const drawParticles = () => {
  if (!ctx) return;
  ctx.clearRect(0, 0, width, height);

  for (let i = 0; i < particles.length; i++) {
    for (let j = i + 1; j < particles.length; j++) {
      const dx = particles[i].x - particles[j].x;
      const dy = particles[i].y - particles[j].y;
      const dist = Math.sqrt(dx * dx + dy * dy);
      if (dist < CONNECT_DIST) {
        ctx.beginPath();
        ctx.moveTo(particles[i].x, particles[i].y);
        ctx.lineTo(particles[j].x, particles[j].y);
        ctx.strokeStyle = `rgba(74, 165, 255, ${
          0.12 * (1 - dist / CONNECT_DIST)
        })`;
        ctx.stroke();
      }
    }
  }

  particles.forEach((p) => {
    ctx.beginPath();
    ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
    ctx.fillStyle = `rgba(154, 201, 255, ${p.opacity})`;
    ctx.fill();

    if (Math.random() < 0.007) {
      ctx.beginPath();
      ctx.arc(p.x - 2, p.y - 2, 1, 0, Math.PI * 2);
      ctx.fillStyle = "rgba(255, 107, 139, 0.7)";
      ctx.fill();
    }
  });
};

const updateParticles = () => {
  particles.forEach((p) => {
    p.x += p.vx;
    p.y += p.vy;
    if (p.x < 0 || p.x > width) p.vx *= -0.95;
    if (p.y < 0 || p.y > height) p.vy *= -0.95;
    p.x = Math.max(0, Math.min(width, p.x));
    p.y = Math.max(0, Math.min(height, p.y));
  });
};

const animate = () => {
  updateParticles();
  drawParticles();
  animationFrame = requestAnimationFrame(animate);
};

const handleResize = () => {
  resizeCanvas();
};

// ---------- 打字机 ----------
const lines = [
  "白发红瞳，星枢演构",
  '柔弱的身躯里，是无边的浩瀚星空',
  "跨越时空的鸿沟，以双手触碰世界的宏大",
  "深空联合研究院 · 星炬学院教授",
  "用科技延伸自身，以计算推演未来",
  "机械义肢之下，藏着温柔的梦想",
  "正因为能看到星星，人类才会对天空之外的世界产生好奇",
  "冷静的外表，炽热的内心",
];

let lineIndex = 0,
  charIndex = 0,
  isDeleting = false,
  typeTimer = null;

const typeEffect = () => {
  const currentLine = lines[lineIndex];
  if (isDeleting) {
    typedText.value = currentLine.substring(0, charIndex - 1);
    charIndex--;
  } else {
    typedText.value = currentLine.substring(0, charIndex + 1);
    charIndex++;
  }

  if (!isDeleting && charIndex === currentLine.length) {
    isDeleting = true;
    typeTimer = setTimeout(typeEffect, 2000);
  } else if (isDeleting && charIndex === 0) {
    isDeleting = false;
    lineIndex = (lineIndex + 1) % lines.length;
    typeTimer = setTimeout(typeEffect, 400);
  } else {
    const speed = isDeleting ? 40 : 100;
    typeTimer = setTimeout(typeEffect, speed);
  }
};

onMounted(() => {
  // 轮播定时器
  imgTimer = setInterval(() => {
    currentIndex.value = (currentIndex.value + 1) % randomFive.value.length;
  }, 5000);

  // 粒子系统
  resizeCanvas();
  animate();
  window.addEventListener("resize", handleResize);

  // 打字机
  typeTimer = setTimeout(typeEffect, 500);
});

onBeforeUnmount(() => {
  if (imgTimer) clearInterval(imgTimer);
  if (animationFrame) cancelAnimationFrame(animationFrame);
  if (typeTimer) clearTimeout(typeTimer);
  window.removeEventListener("resize", handleResize);
});
</script>

<style scoped lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;800&family=Rajdhani:wght@300;500;600&family=Share+Tech+Mono&display=swap");

.morning-home {
  position: relative;
  width: 100vw;
  height: 100vh;
  background: radial-gradient(ellipse at 40% 50%, #0c1424 0%, #03050f 80%);
  overflow: hidden;
  color: #eef5ff;
  font-family: "Rajdhani", "Inter", "PingFang SC", "Microsoft YaHei", sans-serif;
}

/* ===== 轮播样式 ===== */
.carousel {
  position: absolute;
  inset: 0;
  z-index: 0;
  pointer-events: none;

  &::after {
    content: "";
    position: absolute;
    inset: 0;
    background: rgba(10, 12, 22, 0.45);
    backdrop-filter: blur(1px) saturate(120%);
    z-index: 2;
  }

  .carousel-image {
    position: absolute;
    width: 100%;
    height: 100%;
    object-fit: cover;
    opacity: 0;
    transition: opacity 1.2s ease;

    transform: scale(1.02);

    &.active {
      opacity: 1;
      transform: scale(1);
    }
  }
}
.carousel2 {
  display: none;
}

/* ===== 背景网格 ===== */
.grid-bg {
  position: absolute;
  inset: 0;
  background-image: linear-gradient(
      rgba(60, 140, 230, 0.03) 1px,
      transparent 1px
    ),
    linear-gradient(90deg, rgba(60, 140, 230, 0.03) 1px, transparent 1px);
  background-size: 40px 40px;
  z-index: 1;
  opacity: 0.4;
}

.hex-bg {
  position: absolute;
  inset: 0;
  background-image: repeating-linear-gradient(
    45deg,
    rgba(74, 165, 255, 0.02) 0px,
    rgba(74, 165, 255, 0.02) 2px,
    transparent 2px,
    transparent 30px
  );
  z-index: 1;
  pointer-events: none;
}

.particle-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 2;
  pointer-events: none;
  opacity: 0.7;
}

/* 全息圆盘 */
.hologram-disk {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: min(700px, 80vw);
  height: min(700px, 80vw);
  background: radial-gradient(
    circle,
    rgba(30, 100, 200, 0.08) 0%,
    rgba(10, 20, 40, 0.02) 60%,
    transparent 80%
  );
  border-radius: 50%;
  border: 1px solid rgba(74, 165, 255, 0.15);
  box-shadow: 0 0 80px rgba(40, 130, 250, 0.2),
    inset 0 0 40px rgba(74, 165, 255, 0.1);
  z-index: 5;
  pointer-events: none;
  animation: diskRotate 30s linear infinite;
  &::before {
    content: "";
    position: absolute;
    inset: 10%;
    border: 1px dashed rgba(74, 165, 255, 0.2);
    border-radius: 50%;
    opacity: 0.6;
  }
}

@keyframes diskRotate {
  from {
    transform: translate(-50%, -50%) rotate(0deg);
  }
  to {
    transform: translate(-50%, -50%) rotate(360deg);
  }
}

/* 红瞳点缀 */
.red-dot {
  position: absolute;
  width: 4px;
  height: 4px;
  background: #ff6b8b;
  border-radius: 50%;
  filter: blur(2px);
  box-shadow: 0 0 12px #ff6b8b;
  opacity: 0.6;
  animation: redBreath 3s infinite;
  z-index: 15;
  pointer-events: none;
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

/* 机械角 */
.mech-corner {
  position: absolute;
  top: 20px;
  left: 20px;
  width: 60px;
  height: 60px;
  background: radial-gradient(
    circle at 30% 30%,
    rgba(74, 165, 255, 0.2),
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
  z-index: 5;
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

/* 数据流 */
.data-flow {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 3px;
  background: linear-gradient(
    90deg,
    transparent,
    #4aa5ff 20%,
    #9ac9ff 50%,
    #4aa5ff 80%,
    transparent
  );
  filter: blur(2px);
  animation: flowMove 3s linear infinite;
  z-index: 10;
  opacity: 0.7;
}

@keyframes flowMove {
  0% {
    transform: translateX(-100%);
  }
  100% {
    transform: translateX(100%);
  }
}

/* 标题光环 */
.title-ring {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: min(800px, 90vw);
  height: min(800px, 90vw);
  border: 1px solid rgba(74, 165, 255, 0.2);
  border-radius: 50%;
  box-shadow: 0 0 60px rgba(74, 165, 255, 0.2),
    inset 0 0 40px rgba(74, 165, 255, 0.1);
  animation: ringRotate 40s linear infinite;
  z-index: 4;
  pointer-events: none;
}

@keyframes ringRotate {
  from {
    transform: translate(-50%, -50%) rotate(0deg);
  }
  to {
    transform: translate(-50%, -50%) rotate(360deg);
  }
}

/* 浮动粒子 */
.floating-particles {
  position: absolute;
  inset: 0;
  z-index: 6;
  pointer-events: none;
}

.particle {
  position: absolute;
  width: 2px;
  height: 2px;
  background: #9ac9ff;
  border-radius: 50%;
  box-shadow: 0 0 8px #4aa5ff;
  opacity: 0.4;
  animation: floatParticle 8s ease-in-out infinite;
}

@keyframes floatParticle {
  0%,
  100% {
    transform: translate(0, 0);
    opacity: 0.2;
  }
  25% {
    transform: translate(15px, -15px);
    opacity: 0.6;
  }
  50% {
    transform: translate(-10px, 20px);
    opacity: 0.3;
  }
  75% {
    transform: translate(10px, 10px);
    opacity: 0.5;
  }
}

/* 旋转环 */
.rotating-ring {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: min(500px, 60vw);
  height: min(500px, 60vw);
  border: 1px dashed rgba(154, 201, 255, 0.15);
  border-radius: 50%;
  animation: ringRotateReverse 30s linear infinite;
  z-index: 4;
  pointer-events: none;
}

@keyframes ringRotateReverse {
  from {
    transform: translate(-50%, -50%) rotate(0deg);
  }
  to {
    transform: translate(-50%, -50%) rotate(-360deg);
  }
}

/* 主内容区 */
.content {
  position: relative;
  z-index: 20;
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
  padding: 0 20px;
  pointer-events: none;
}

.title-backdrop {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 110%;
  max-width: 1200px;
  height: 220px;
  background: radial-gradient(
    ellipse at center,
    rgba(10, 20, 40, 0.7) 0%,
    rgba(5, 10, 20, 0.4) 70%
  );
  backdrop-filter: blur(10px);
  border-radius: 60px;
  border: 1px solid rgba(74, 165, 255, 0.3);
  box-shadow: 0 0 80px rgba(0, 30, 70, 0.8),
    inset 0 0 40px rgba(74, 165, 255, 0.2);
  z-index: -1;
}

.main-title {
  font-family: "Orbitron", sans-serif;
  font-size: clamp(3.5rem, 12vw, 6rem);
  font-weight: 800;
  letter-spacing: 10px;
  text-transform: uppercase;
  background: linear-gradient(135deg, #ffffff, #b0e0ff, #5a9eff);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  text-shadow: 0 0 30px rgba(90, 158, 255, 0.8),
    0 0 60px rgba(90, 158, 255, 0.4);
  margin-bottom: 20px;
  animation: titleGlow 2s infinite alternate, titleFloat 4s ease-in-out infinite;
  position: relative;
  z-index: 21;
  -webkit-text-stroke: 2px #5a9eff;
}

@keyframes titleGlow {
  0% {
    text-shadow: 0 0 30px rgba(90, 158, 255, 0.6),
      0 0 60px rgba(90, 158, 255, 0.3);
  }
  100% {
    text-shadow: 0 0 50px rgba(120, 200, 255, 1),
      0 0 90px rgba(120, 200, 255, 0.7);
  }
}

@keyframes titleFloat {
  0%,
  100% {
    transform: translateY(0) scale(1);
  }
  50% {
    transform: translateY(-5px) scale(1.02);
  }
}

.title-brackets {
  position: relative;
  display: inline-block;

  &::before,
  &::after {
    content: "【";
    position: absolute;
    font-family: "Orbitron", sans-serif;
    font-size: clamp(3.5rem, 12vw, 6rem);
    color: rgba(154, 201, 255, 0.9);
    left: -1em;
    top: 0;
    animation: bracketPulse 2s infinite, bracketGlow 3s infinite;
    text-shadow: 0 0 20px #3f8aff;
  }

  &::after {
    content: "】";
    left: auto;
    right: -1em;
  }
}

@keyframes bracketPulse {
  0%,
  100% {
    opacity: 0.6;
  }
  50% {
    opacity: 1;
  }
}

@keyframes bracketGlow {
  0% {
    filter: drop-shadow(0 0 5px #3f8aff);
  }
  50% {
    filter: drop-shadow(0 0 20px #7abfff);
  }
  100% {
    filter: drop-shadow(0 0 5px #3f8aff);
  }
}

.subtitle-area {
  font-size: 1.8rem;
  min-height: 3rem;
  margin-top: 20px;
  font-family: "Share Tech Mono", monospace;
  color: #ffffff;
  text-shadow: 0 0 15px #4aa5ff, 0 0 30px #2a6eb0;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  background: rgba(0, 5, 15, 0.5);
  backdrop-filter: blur(8px);
  padding: 12px 32px;
  border-radius: 60px;
  border: 1px solid rgba(74, 165, 255, 0.5);
  box-shadow: 0 0 40px rgba(30, 100, 200, 0.4);
  z-index: 21;
}

.cursor {
  width: 12px;
  height: 2.2rem;
  background: #4aa5ff;
  border-radius: 2px;
  animation: blink 1s step-end infinite;
  box-shadow: 0 0 15px #4aa5ff;
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

.footer-info {
  position: absolute;
  bottom: 30px;
  left: 0;
  right: 0;
  text-align: center;
  z-index: 20;
  font-family: "Rajdhani", sans-serif;
  font-weight: 300;
  letter-spacing: 2px;
  color: rgba(200, 225, 255, 0.9);
  text-shadow: 0 0 15px #2a6eb0;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(74, 165, 255, 0.2),
    transparent
  );
  padding: 15px 0;
  backdrop-filter: blur(8px);
  border-top: 1px solid rgba(74, 165, 255, 0.3);
  border-bottom: 1px solid rgba(74, 165, 255, 0.3);
  width: 75%;
  margin: 0 auto;
  border-radius: 40px;
  transform: translateX(-50%);
  left: 50%;
  bottom: 20px;
  white-space: nowrap;
  overflow: hidden;

  span {
    display: inline-block;
    animation: scrollText 14s linear infinite;
  }
}

@keyframes scrollText {
  0% {
    transform: translateX(100%);
  }
  20% {
    transform: translateX(0);
  }
  80% {
    transform: translateX(0);
  }
  100% {
    transform: translateX(-100%);
  }
}

/* 移动端适配 */
@media (max-width: 700px) {
  .carousel1 {
    display: none;
  }
  .carousel2 {
    display: block;
  }
  .main-title {
    font-size: 2.8rem;
    letter-spacing: 4px;
  }
  .title-brackets::before,
  .title-brackets::after {
    font-size: 2.8rem;
  }
  .subtitle-area {
    font-size: 1.2rem;
    padding: 8px 16px;
  }
  .footer-info {
    width: 90%;
    font-size: 0.8rem;
  }
  .title-backdrop {
    height: 160px;
  }
}
</style>
