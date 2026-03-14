<template>
  <header class="app-header">
    <!-- 粒子背景 canvas（动态科技感） -->
    <canvas ref="particleCanvas" class="particle-canvas"></canvas>

    <h1 class="title">莫宁·星枢演构</h1>
    <!-- 在线人数展示 -->
    <div class="online-count" v-if="onlineCount !== null">
      当前在线：<span class="count">{{ onlineCount }}人</span>
    </div>
    <!-- 移动端汉堡按钮 -->
    <button
      class="hamburger"
      @click="toggleMobileNav"
      aria-label="Toggle navigation"
    >
      <span :class="{ open: mobileNavOpen }"></span>
      <span :class="{ open: mobileNavOpen }"></span>
      <span :class="{ open: mobileNavOpen }"></span>
    </button>

    <!-- 普通导航 & 移动端下拉导航 -->
    <nav :class="['nav-links', { 'mobile-open': mobileNavOpen }]">
      <RouterLink
        v-for="(item, index) in navItems"
        :key="item.name"
        :to="item.path"
        class="nav-item"
        active-class="active-link"
        @click="mobileNavOpen = false"
      >
        <span class="item-text">{{ item.name }}</span>
      </RouterLink>

      <a
        href="https://slty.site/#/redirector"
        target="_blank"
        rel="noopener"
        class="nav-item"
        active-class="active-link"
        @click="mobileNavOpen = false"
      >
        <span class="item-text">霜落映界</span>
      </a>
    </nav>

    <!-- 装饰性元素：数据流光带 + 扫描线 -->
    <div class="data-stream"></div>
    <div class="scan-line"></div>
  </header>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, nextTick } from "vue";
import { io } from "socket.io-client";

const navItems = [
  { name: "演构中枢", path: "/" },
  { name: "时序星轨", path: "/timeLine" },
  { name: "信号回响", path: "/message" },
  { name: "视觉演算", path: "/gallery" },
  { name: "共享节点", path: "/resources" },
  { name: "智械接口", path: "/talk" },
  { name: "微械终端", path: "/game" },
  { name: "声波档案", path: "/music" },
  { name: "文献库", path: "/wiki" },
  // { name: "致谢名录", path: "/thanks" },
];

const mobileNavOpen = ref(false);
function toggleMobileNav() {
  mobileNavOpen.value = !mobileNavOpen.value;
}

const siteId = "moning";

const onlineCount = ref<number | null>(null);

const socket: any = io(import.meta.env.VITE_API_BASE_URL, {
  query: { siteId },
});

onMounted(() => {
  socket.on("onlineCount", (count: number) => {
    onlineCount.value = count;
  });

  // 初始化粒子背景
  initParticles();
});

onBeforeUnmount(() => {
  socket.disconnect();
  if (animationFrame) {
    cancelAnimationFrame(animationFrame);
  }
});

// 粒子系统
const particleCanvas = ref<HTMLCanvasElement | null>(null);
let ctx: CanvasRenderingContext2D | null = null;
let particles: Array<{
  x: number;
  y: number;
  vx: number;
  vy: number;
  size: number;
  opacity: number;
}> = [];
let animationFrame: number;

const initParticles = () => {
  nextTick(() => {
    if (!particleCanvas.value) return;

    const canvas = particleCanvas.value;
    ctx = canvas.getContext("2d");

    const resize = () => {
      canvas.width = window.innerWidth;
      canvas.height = 64; // 和header同高
    };

    window.addEventListener("resize", resize);
    resize();

    // 初始化粒子
    const particleCount = 40;
    for (let i = 0; i < particleCount; i++) {
      particles.push({
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        vx: (Math.random() - 0.5) * 0.3,
        vy: (Math.random() - 0.5) * 0.1,
        size: Math.random() * 1.5 + 0.5,
        opacity: Math.random() * 0.3 + 0.1,
      });
    }

    const animate = () => {
      if (!ctx || !canvas) return;

      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // 绘制连接线
      ctx.strokeStyle = "rgba(74, 165, 255, 0.1)";
      ctx.lineWidth = 0.5;

      for (let i = 0; i < particles.length; i++) {
        for (let j = i + 1; j < particles.length; j++) {
          const dx = particles[i].x - particles[j].x;
          const dy = particles[i].y - particles[j].y;
          const distance = Math.sqrt(dx * dx + dy * dy);

          if (distance < 80) {
            ctx.beginPath();
            ctx.moveTo(particles[i].x, particles[i].y);
            ctx.lineTo(particles[j].x, particles[j].y);
            ctx.strokeStyle = `rgba(74, 165, 255, ${
              0.05 * (1 - distance / 80)
            })`;
            ctx.stroke();
          }
        }
      }

      // 绘制粒子
      particles.forEach((p) => {
        ctx!.beginPath();
        ctx!.arc(p.x, p.y, p.size, 0, Math.PI * 2);
        ctx!.fillStyle = `rgba(154, 201, 255, ${p.opacity})`;
        ctx!.fill();

        // 移动
        p.x += p.vx;
        p.y += p.vy;

        // 边界重置
        if (p.x < 0) p.x = canvas.width;
        if (p.x > canvas.width) p.x = 0;
        if (p.y < 0) p.y = canvas.height;
        if (p.y > canvas.height) p.y = 0;
      });

      animationFrame = requestAnimationFrame(animate);
    };

    animate();
  });
};
</script>

<style scoped lang="scss">
/* 莫宁 — 蓝白机械基调，红瞳仅作微光点缀，增加精致装饰 */
.app-header {
  // ---------- 配色系统 ----------
  --space-bg: #0a0c16; // 深空底色
  --panel-bg: #121826; // 面板深色
  --accent-blue: #4aa5ff; // 冰蓝主色
  --accent-light: #9ac9ff; // 浅蓝高光
  --red-pupil: #ff6b8b; // 低饱和度红瞳
  --mech-grid: rgba(74, 165, 255, 0.08); // 半透明网格
  --text-primary: #f0f6ff; // 冷白文字

  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  height: 64px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 40px;

  // 背景：深色+蓝色光晕+机械网格
  background: radial-gradient(
      circle at 80% 30%,
      rgba(74, 165, 255, 0.08) 0%,
      transparent 50%
    ),
    linear-gradient(135deg, var(--space-bg) 0%, var(--panel-bg) 100%);

  // 精细六边形网格 + 点阵装饰
  background-image: repeating-linear-gradient(
      45deg,
      var(--mech-grid) 0px,
      var(--mech-grid) 1px,
      transparent 1px,
      transparent 12px
    ),
    repeating-linear-gradient(
      135deg,
      var(--mech-grid) 0px,
      var(--mech-grid) 1px,
      transparent 1px,
      transparent 12px
    ),
    radial-gradient(
      circle at 20% 30%,
      rgba(74, 165, 255, 0.03) 1px,
      transparent 1px
    ),
    radial-gradient(
      circle at 80% 70%,
      rgba(154, 201, 255, 0.03) 1px,
      transparent 1px
    );
  background-size: auto, auto, 20px 20px, 30px 30px;
  background-blend-mode: overlay;
  backdrop-filter: blur(10px) saturate(180%);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.5),
    0 0 0 1px rgba(74, 165, 255, 0.2) inset, 0 4px 12px rgba(74, 165, 255, 0.1);
  border-bottom: 1px solid rgba(74, 165, 255, 0.25);
  animation: slideInDown 0.4s ease-out;

  // 粒子画布
  .particle-canvas {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
    z-index: 1;
    opacity: 0.5;
  }

  // 数据流光带 (流动的科技感)
  .data-stream {
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 1px;
    background: linear-gradient(
      90deg,
      transparent 0%,
      rgba(74, 165, 255, 0.5) 20%,
      rgba(154, 201, 255, 0.8) 50%,
      rgba(74, 165, 255, 0.5) 80%,
      transparent 100%
    );
    filter: blur(1px);
    animation: data-stream-flow 3s linear infinite;
    z-index: 2;
    opacity: 0.6;
  }

  // 扫描线效果
  .scan-line {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 2px;
    background: linear-gradient(
      90deg,
      transparent,
      rgba(154, 201, 255, 0.3) 50%,
      transparent
    );
    filter: blur(2px);
    animation: scan-move 4s ease-in-out infinite;
    z-index: 2;
    opacity: 0.4;
  }

  // 右侧机械义肢轮廓增强
  &::before {
    content: "";
    position: absolute;
    right: 2%;
    top: 5%;
    width: 300px;
    height: 80px;
    background: linear-gradient(
        120deg,
        transparent 20%,
        rgba(74, 165, 255, 0.12) 40%,
        rgba(154, 201, 255, 0.08) 60%,
        transparent 80%
      ),
      repeating-linear-gradient(
        -10deg,
        rgba(255, 255, 255, 0.02) 0px,
        rgba(255, 255, 255, 0.02) 6px,
        transparent 6px,
        transparent 16px
      );
    filter: blur(8px);
    transform: skewX(-8deg) rotate(1deg);
    animation: mech-float 8s ease-in-out infinite;
    mix-blend-mode: overlay;
    pointer-events: none;
    z-index: 2;
  }

  // 左上角六边形装饰（模拟机械结构）
  &::after {
    content: "";
    position: absolute;
    left: 20px;
    top: 12px;
    width: 40px;
    height: 40px;
    background: radial-gradient(
        circle at 30% 30%,
        rgba(74, 165, 255, 0.15) 0%,
        transparent 60%
      ),
      repeating-linear-gradient(
        60deg,
        rgba(74, 165, 255, 0.1) 0px,
        rgba(74, 165, 255, 0.1) 2px,
        transparent 2px,
        transparent 8px
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
    opacity: 0.5;
    filter: blur(1px);
    animation: rotate-slow 20s linear infinite;
    pointer-events: none;
    z-index: 2;
  }

  .title {
    position: relative;
    font-family: "Orbitron", "Cinzel", "Noto Serif SC", serif;
    font-size: 26px;
    font-weight: 600;
    letter-spacing: 3px;
    background: linear-gradient(135deg, #fff, var(--accent-light), #fff);
    background-clip: text;
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    text-shadow: 0 0 20px rgba(74, 165, 255, 0.4);
    transition: transform 0.2s, text-shadow 0.2s;
    animation: title-float 3s infinite alternate ease-in-out;
    z-index: 3;

    &::before {
      content: "【";
      position: absolute;
      left: -20px;
      top: 0;
      color: rgba(74, 165, 255, 0.5);
      font-size: 24px;
      font-weight: 300;
      animation: bracket-pulse 2s infinite;
    }

    &::after {
      content: "】";
      position: absolute;
      right: -20px;
      top: 0;
      color: rgba(74, 165, 255, 0.5);
      font-size: 24px;
      font-weight: 300;
      animation: bracket-pulse 2s infinite 1s;
    }

    &:hover {
      transform: translateY(-2px);
      text-shadow: 0 0 30px rgba(74, 165, 255, 0.7);
    }
  }

  .online-count {
    position: relative;
    margin-left: 16px;
    padding: 5px 18px 5px 16px;
    font-family: "Share Tech Mono", "Fira Code", monospace;
    font-size: 0.9rem;
    color: var(--text-primary);
    background: rgba(10, 12, 22, 0.7);
    border: 1px solid rgba(74, 165, 255, 0.3);
    border-radius: 40px;
    backdrop-filter: blur(4px);
    box-shadow: 0 0 12px rgba(74, 165, 255, 0.2);
    cursor: default;
    transition: 0.2s;
    z-index: 3;
    overflow: hidden;

    &::before {
      content: "";
      position: absolute;
      top: -50%;
      left: -50%;
      width: 200%;
      height: 200%;
      background: linear-gradient(
        45deg,
        transparent 30%,
        rgba(154, 201, 255, 0.1) 50%,
        transparent 70%
      );
      animation: shine 3s infinite;
      pointer-events: none;
    }

    &:hover {
      border-color: var(--accent-blue);
      box-shadow: 0 0 20px rgba(74, 165, 255, 0.5);
    }

    .count {
      display: inline-block;
      margin-left: 6px;
      font-weight: 600;
      background: linear-gradient(90deg, var(--accent-blue), #bde0ff);
      background-clip: text;
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      text-shadow: 0 0 8px currentColor;
      position: relative;
    }
  }

  .nav-links {
    display: flex;
    gap: 16px;
    align-items: center;
    z-index: 3;

    .nav-item {
      position: relative;
      color: var(--text-primary);
      font-weight: 500;
      text-decoration: none;
      padding: 6px 12px 6px 8px;
      font-family: "Rajdhani", "Inter", "PingFang SC", sans-serif;
      font-size: 1rem;
      letter-spacing: 0.5px;
      transition: color 0.2s, transform 0.2s, background 0.2s;
      display: flex;
      align-items: center;
      gap: 8px;
      border-radius: 4px;
      overflow: hidden;

      .item-text {
        position: relative;
      }

      // 冰蓝能量下划线 (增强版)
      &::after {
        content: "";
        position: absolute;
        left: 50%;
        bottom: 2px;
        width: 0;
        height: 2px;
        background: linear-gradient(
          90deg,
          transparent,
          var(--accent-blue) 20%,
          var(--accent-light) 80%,
          transparent
        );
        border-radius: 2px;
        transform: translateX(-50%);
        transition: width 0.25s ease;
        box-shadow: 0 0 8px var(--accent-blue);
      }

      // 背景光晕 (hover时出现)
      &::before {
        content: "";
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background: radial-gradient(
          circle at center,
          rgba(74, 165, 255, 0.1),
          transparent 70%
        );
        opacity: 0;
        transition: opacity 0.2s;
        pointer-events: none;
        border-radius: 4px;
      }

      // 红瞳点缀（极微，仅active可见）
      .red-dot {
        display: none;
      }

      &:hover {
        color: var(--accent-light);
        transform: translateY(-1px);
        background: rgba(74, 165, 255, 0.05);
      }

      &:hover::after {
        width: 80%;
      }

      &:hover::before {
        opacity: 1;
      }
    }

    .active-link {
      color: var(--accent-light);
      font-weight: 600;
      background: rgba(74, 165, 255, 0.08);

      // 下划线常亮
      &::after {
        width: 90%;
        background: linear-gradient(
          90deg,
          transparent,
          var(--accent-blue) 10%,
          var(--accent-light) 90%,
          transparent
        );
        box-shadow: 0 0 16px var(--accent-blue);
        height: 3px;
      }

      &::before {
        opacity: 0.5;
      }
    }
  }

  .hamburger {
    display: none;
    flex-direction: column;
    justify-content: space-around;
    width: 28px;
    height: 24px;
    background: none;
    border: none;
    cursor: pointer;
    padding: 0;
    z-index: 3;

    span {
      display: block;
      width: 100%;
      height: 3px;
      background: var(--accent-blue);
      border-radius: 2px;
      transition: 0.3s;
      box-shadow: 0 0 8px var(--accent-blue);
    }

    span.open:nth-child(1) {
      transform: translateY(10px) rotate(45deg);
      background: var(--accent-light);
      box-shadow: 0 0 12px var(--accent-light);
    }

    span.open:nth-child(2) {
      opacity: 0;
    }

    span.open:nth-child(3) {
      transform: translateY(-10px) rotate(-45deg);
      background: var(--accent-light);
      box-shadow: 0 0 12px var(--accent-light);
    }
  }

  @media (max-width: 768px) {
    padding: 0 20px;

    .title {
      display: none;
    }

    .online-count {
      margin-left: 0;
      margin-right: auto;
    }

    .hamburger {
      display: flex;
    }

    .nav-links {
      position: absolute;
      top: 64px;
      left: 0;
      right: 0;
      flex-direction: column;
      background: rgba(10, 12, 22, 0.98);
      backdrop-filter: blur(24px);
      gap: 0;
      max-height: 0;
      overflow: hidden;
      transition: max-height 0.35s ease;
      border-top: 1px solid rgba(74, 165, 255, 0.25);
      border-bottom: 1px solid rgba(74, 165, 255, 0.25);

      &.mobile-open {
        max-height: 520px;
      }

      .nav-item {
        padding: 16px 24px;
        width: 100%;
        border-bottom: 1px solid rgba(255, 255, 255, 0.03);
        justify-content: space-between;

        &::after {
          bottom: 8px;
          width: 0 !important;
        }

        &:hover::after {
          width: 90% !important;
        }
      }
    }

    // 移动端隐藏部分装饰避免干扰
    &::before {
      opacity: 0.2;
    }

    .data-stream {
      opacity: 0.3;
    }
  }
}

/* 新增动画 */
@keyframes data-stream-flow {
  0% {
    transform: translateX(-100%);
  }
  100% {
    transform: translateX(100%);
  }
}

@keyframes scan-move {
  0% {
    top: 0;
    opacity: 0.2;
  }
  50% {
    top: 100%;
    opacity: 0.6;
  }
  100% {
    top: 0;
    opacity: 0.2;
  }
}

@keyframes shine {
  0% {
    transform: translateX(-100%) rotate(45deg);
  }
  20%,
  100% {
    transform: translateX(100%) rotate(45deg);
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

@keyframes rotate-slow {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

@keyframes bracket-pulse {
  0%,
  100% {
    opacity: 0.3;
  }
  50% {
    opacity: 0.8;
  }
}

@keyframes index-pulse {
  0%,
  100% {
    border-color: rgba(255, 107, 139, 0.5);
    color: rgba(255, 107, 139, 0.9);
  }
  50% {
    border-color: rgba(255, 107, 139, 0.9);
    color: rgba(255, 107, 139, 1);
  }
}

@keyframes pupil-breathe {
  0%,
  100% {
    opacity: 0.8;
    transform: scale(1);
  }
  50% {
    opacity: 1;
    transform: scale(1.2);
  }
}

@keyframes slideInDown {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes title-float {
  0% {
    transform: translateY(0);
    text-shadow: 0 0 16px rgba(74, 165, 255, 0.3);
  }
  100% {
    transform: translateY(-3px);
    text-shadow: 0 0 28px rgba(74, 165, 255, 0.7);
  }
}

@keyframes mech-float {
  0% {
    opacity: 0.2;
    transform: skewX(-8deg) rotate(1deg) translateX(0);
  }
  50% {
    opacity: 0.5;
    transform: skewX(-8deg) rotate(2deg) translateX(-8px);
  }
  100% {
    opacity: 0.2;
    transform: skewX(-8deg) rotate(1deg) translateX(0);
  }
}
</style>
