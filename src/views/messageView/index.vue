<template>
  <div class="morning-message-board">
    <!-- 背景轮播（冷色滤镜） -->
    <div class="carousel carousel1" aria-hidden="true">
      <img
        v-for="(src, idx) in randomFive"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>
    <div class="carousel carousel2" aria-hidden="true">
      <img
        v-for="(src, idx) in randomFive2"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>

    <div class="data-flow"></div>

    <!-- 半透明顶部标题 -->
    <header class="board-header" role="banner">
      <div class="title-wrap">
        <h1>演构回响</h1>
        <span class="title-count">（共{{ total }}条）</span>
        <p class="subtitle">星枢演构 · 信号捕获</p>
      </div>
    </header>

    <!-- 留言展示区（无限滚动） -->
    <section class="message-list" ref="listContainer">
      <div class="message-list-inner">
        <transition-group name="msg" tag="div">
          <div
            v-for="(msg, idx) in messages"
            :key="msg.id || msg._tempId || idx"
            class="message-card"
            :data-index="idx"
            tabindex="0"
            role="article"
            :aria-label="`留言来自 ${msg.name || '匿名'}，内容：${msg.content}`"
          >
            <div class="message-meta">
              <div class="left-meta">
                <div class="name-avatar" :title="msg.name || '匿名'">
                  {{ getInitial(msg.name) }}
                </div>
                <div class="meta-texts">
                  <div class="message-name">{{ msg.name || "匿名" }}</div>
                  <div class="message-time">
                    {{ formatTime(msg.created_at) }}
                  </div>
                </div>
              </div>
            </div>
            <p class="message-content">{{ msg.content }}</p>
          </div>
        </transition-group>

        <!-- 哨兵元素 + 加载提示 -->
        <div ref="sentinel" class="sentinel"></div>
        <div v-if="loadingMore" class="loading-more">
          <span class="loader"></span> 演算中...
        </div>
        <div v-if="!hasMore && messages.length > 0" class="no-more">
          —— 已捕获全部信号 ——
        </div>
      </div>
    </section>

    <!-- 底部发送区 -->
    <section class="message-form" aria-label="发送信号">
      <label class="sr-only" for="mb-name">昵称</label>
      <input
        id="mb-name"
        v-model="name"
        type="text"
        placeholder="昵称 (可选)"
        @keydown.enter.prevent
      />

      <label class="sr-only" for="mb-content">信号内容</label>
      <textarea
        id="mb-content"
        v-model="content"
        placeholder="输入信号内容..."
        @keydown.ctrl.enter.prevent="submitMessage"
        @input="autoGrow"
        ref="textareaRef"
      />

      <div class="form-row">
        <div class="hint"><kbd>Ctrl</kbd> + <kbd>Enter</kbd> 发送</div>
        <button @click="submitMessage" :disabled="isSending || !content.trim()">
          <span v-if="!isSending">发送信号</span>
          <span v-else>发送中…</span>
        </button>
      </div>
    </section>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick, onBeforeUnmount } from "vue";
import { getMessageList, createMessage } from "@/api/modules/message";

// ========== 数据状态 ==========
const messages = ref<any[]>([]);
const total = ref(0);
const name = ref(localStorage.getItem("message_name") || "");
const content = ref("");
const isSending = ref(false);
const textareaRef = ref<HTMLTextAreaElement | null>(null);

// 无限滚动相关
const listContainer = ref<HTMLElement | null>(null);
const sentinel = ref<HTMLElement | null>(null);
const currentPage = ref(1);
const pageSize = 20;
const hasMore = ref(true);
const loadingMore = ref(false);
let observer: IntersectionObserver | null = null;

// ========== 获取留言（分页） ==========
const fetchMessages = async (page: number, append = false) => {
  if (loadingMore.value) return;
  loadingMore.value = true;
  try {
    const res = await getMessageList({ page, pageSize });
    const newMessages = res.data || [];
    total.value = res.pagination.total;

    if (append) {
      messages.value = [...messages.value, ...newMessages];
    } else {
      messages.value = newMessages;
    }

    // 判断是否还有更多
    hasMore.value = messages.value.length < total.value;
  } catch (err) {
    console.error(err);
  } finally {
    loadingMore.value = false;
  }
};

// 加载下一页
const loadMore = () => {
  if (!hasMore.value || loadingMore.value) return;
  currentPage.value++;
  fetchMessages(currentPage.value, true);
};

// ========== 提交留言 ==========
const submitMessage = async () => {
  if (!content.value.trim() || isSending.value) return;
  isSending.value = true;
  const payload = { name: name.value || "匿名", content: content.value };
  try {
    localStorage.setItem("message_name", name.value);
    content.value = "";
    await nextTick();
    await createMessage(payload);
    // 重置到第一页，重新加载
    currentPage.value = 1;
    await fetchMessages(1, false);
    // 滚动到顶部（可选）
    listContainer.value?.scrollTo({ top: 0, behavior: "smooth" });
  } catch (err) {
    console.error(err);
  } finally {
    isSending.value = false;
  }
};

// ========== 工具函数 ==========
const formatTime = (time: string) => {
  if (!time) return "";
  const d = new Date(time);
  const y = d.getFullYear();
  const m = String(d.getMonth() + 1).padStart(2, "0");
  const day = String(d.getDate()).padStart(2, "0");
  const hh = String(d.getHours()).padStart(2, "0");
  const mm = String(d.getMinutes()).padStart(2, "0");
  return `${y}-${m}-${day} ${hh}:${mm}`;
};

const getInitial = (n?: string) => {
  if (!n) return "匿";
  return n.trim().slice(0, 1).toUpperCase();
};

const autoGrow = () => {
  const ta = textareaRef.value;
  if (!ta) return;
  ta.style.height = "auto";
  ta.style.height = Math.min(ta.scrollHeight, 220) + "px";
};

// ========== 背景轮播 ==========
const modules = import.meta.glob("@/assets/images1/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const allSrcs: string[] = Object.values(modules).map((mod: any) => mod.default);
const modules2 = import.meta.glob("@/assets/images2/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const allSrcs2: string[] = Object.values(modules2).map(
  (mod: any) => mod.default
);

function shuffle<T>(arr: T[]): T[] {
  const a = arr.slice();
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}
const randomFive = ref<string[]>(shuffle(allSrcs).slice(0, 5));
const randomFive2 = ref<string[]>(shuffle(allSrcs2).slice(0, 5));
const currentIndex = ref(0);
let imgTimer: number | undefined;

// ========== 初始化 & 监听 ==========
onMounted(async () => {
  // 获取第一页数据
  await fetchMessages(1, false);

  // 轮播定时器
  imgTimer = window.setInterval(() => {
    currentIndex.value =
      (currentIndex.value + 1) % Math.max(1, randomFive.value.length);
  }, 5200);

  // 自动调整文本框高度
  nextTick(() => autoGrow());

  // 创建 IntersectionObserver 监听哨兵
  observer = new IntersectionObserver(
    (entries) => {
      if (entries[0].isIntersecting && hasMore.value && !loadingMore.value) {
        loadMore();
      }
    },
    { threshold: 0.1, rootMargin: "100px" } // 提前100px触发
  );

  if (sentinel.value) {
    observer.observe(sentinel.value);
  }
});

onBeforeUnmount(() => {
  if (imgTimer) clearInterval(imgTimer);
  if (observer) observer.disconnect();
});
</script>

<style scoped lang="scss">
/* 莫宁配色：深空机械 / 冰蓝光效 / 红瞳点缀（极微量） */
$bg-deep: #0a0c16;
$bg-panel: #121826;
$accent-blue: #4aa5ff;
$accent-light: #9ac9ff;
$red-pupil: #ff6b8b;
$text-primary: #f0f6ff;

$card-bg: rgba(10, 15, 30, 0.4);
$card-border: rgba(74, 165, 255, 0.15);
$soft-shadow: 0 10px 30px rgba(0, 0, 0, 0.6);

.morning-message-board {
  position: relative;
  height: 100vh;
  padding-top: 110px;
  display: flex;
  flex-direction: column;
  background: radial-gradient(ellipse at 30% 40%, #0e1424, #03050f);
  font-family: "Rajdhani", "Inter", "PingFang SC", sans-serif;
  color: $text-primary;
  overflow: hidden;

  /* 轮播背景 */
  .carousel {
    position: absolute;
    inset: 0;
    z-index: 0;
    pointer-events: none;

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

  
  /* 数据流 */
  .data-flow {
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 12px;
    background: linear-gradient(
      90deg,
      transparent,
      $accent-blue 20%,
      $accent-light 50%,
      $accent-blue 80%,
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

  /* ---------- 头部 ---------- */
  .board-header {
    position: absolute;
    top: 72px;
    left: 50%;
    transform: translateX(-50%);
    width: calc(100% - 32px);
    max-width: 960px;
    padding: 14px 24px;
    border-radius: 40px;
    background: rgba(10, 15, 30, 0.5);
    backdrop-filter: blur(2px);
    border: 1px solid rgba(74, 165, 255, 0.25);
    box-shadow: 0 0 30px rgba(74, 165, 255, 0.2);
    z-index: 10;

    .title-wrap {
      display: flex;
      align-items: center;
      gap: 16px;

      h1 {
        margin: 0;
        font-family: "Orbitron", sans-serif;
        font-size: 1.8rem;
        font-weight: 600;
        letter-spacing: 2px;
        background: linear-gradient(135deg, #fff, $accent-light);
        background-clip: text;
        -webkit-background-clip: text;
        color: transparent;
        text-shadow: 0 0 15px $accent-blue;
      }

      .title-count {
        font-size: 0.9rem;
        color: $accent-light;
        background: rgba(0, 0, 0, 0.3);
        padding: 2px 10px;
        border-radius: 20px;
        border: 1px solid rgba(74, 165, 255, 0.3);
      }

      .subtitle {
        margin-left: auto;
        font-size: 1rem;
        color: rgba(240, 246, 255, 0.8);
        text-shadow: 0 0 8px $accent-blue;
      }
    }
  }

  /* ---------- 留言列表 (可滚动) ---------- */
  .message-list {
    position: relative;
    z-index: 5;
    flex: 1;
    overflow-y: auto;
    padding: 28px 20px 240px;
    margin-top: 20px;

    .message-list-inner {
      max-width: 960px;
      margin: 0 auto;
      display: flex;
      flex-direction: column;
      gap: 16px;
    }

    .message-card {
      background: $card-bg;
      backdrop-filter: blur(2px);
      border-radius: 20px;
      padding: 18px 20px;
      border: 1px solid $card-border;
      box-shadow: $soft-shadow, inset 0 0 20px rgba(74, 165, 255, 0.1);
      transition: transform 0.2s, box-shadow 0.2s;
      margin-top: 12px;
      &:hover {
        transform: translateY(-4px);
        box-shadow: 0 20px 40px rgba(0, 0, 0, 0.8),
          inset 0 0 30px rgba(74, 165, 255, 0.2);
        border-color: rgba(74, 165, 255, 0.4);
      }

      .message-meta {
        display: flex;
        align-items: center;
        margin-bottom: 12px;

        .left-meta {
          display: flex;
          align-items: center;
          gap: 12px;

          .name-avatar {
            width: 48px;
            height: 48px;
            border-radius: 50%;
            background: linear-gradient(135deg, $accent-blue, $accent-light);
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: bold;
            font-size: 1.2rem;
            color: $bg-deep;
            box-shadow: 0 0 20px $accent-blue;
            text-transform: uppercase;
          }

          .meta-texts {
            .message-name {
              font-weight: 600;
              color: $accent-light;
              font-size: 1.1rem;
              line-height: 1.2;
            }
            .message-time {
              font-size: 0.8rem;
              color: rgba(240, 246, 255, 0.6);
            }
          }
        }
      }

      .message-content {
        margin: 0;
        font-size: 1rem;
        line-height: 1.6;
        white-space: pre-wrap;
        word-break: break-word;
        color: rgba(255, 255, 255, 0.9);
        padding-left: 60px; // 与头像对齐
      }
    }

    /* 哨兵 & 加载提示 */
    .sentinel {
      height: 20px;
      width: 100%;
    }

    .loading-more,
    .no-more {
      text-align: center;
      padding: 20px;
      font-family: "Share Tech Mono", monospace;
      color: $accent-light;
      text-shadow: 0 0 10px currentColor;
      letter-spacing: 1px;

      .loader {
        display: inline-block;
        width: 16px;
        height: 16px;
        border: 2px solid rgba(74, 165, 255, 0.3);
        border-top-color: $accent-light;
        border-radius: 50%;
        animation: spin 0.6s linear infinite;
        margin-right: 8px;
        vertical-align: middle;
      }
    }

    .no-more {
      opacity: 0.6;
      font-size: 0.9rem;
    }
  }

  @keyframes spin {
    to {
      transform: rotate(360deg);
    }
  }

  /* ---------- 底部输入区 ---------- */
  .message-form {
    position: fixed;
    left: 50%;
    transform: translateX(-50%);
    bottom: 0px;
    width: calc(100% - 32px);
    max-width: 960px;
    background: rgba(10, 15, 30, 0.8);
    backdrop-filter: blur(16px);
    padding: 20px;
    border-radius: 30px;
    border: 1px solid rgba(74, 165, 255, 0.3);
    box-shadow: 0 0 50px rgba(74, 165, 255, 0.2),
      inset 0 0 30px rgba(74, 165, 255, 0.1);
    z-index: 20;

    input,
    textarea {
      width: 100%;
      padding: 14px 18px;
      margin-bottom: 12px;
      background: rgba(0, 0, 0, 0.4);
      border: 1px solid rgba(74, 165, 255, 0.3);
      border-radius: 40px;
      font-size: 1rem;
      color: $text-primary;
      outline: none;
      transition: border-color 0.2s, box-shadow 0.2s;

      &:focus {
        border-color: $accent-blue;
        box-shadow: 0 0 20px $accent-blue;
      }

      &::placeholder {
        color: rgba(240, 246, 255, 0.4);
      }
    }

    textarea {
      border-radius: 24px;
      resize: none;
      min-height: 80px;
      line-height: 1.5;
    }

    .form-row {
      display: flex;
      align-items: center;
      justify-content: space-between;

      .hint {
        color: rgba(240, 246, 255, 0.7);
        font-size: 0.9rem;
        kbd {
          background: rgba(0, 0, 0, 0.5);
          border-radius: 6px;
          padding: 2px 8px;
          border: 1px solid rgba(74, 165, 255, 0.4);
          color: $accent-light;
          font-family: "Share Tech Mono", monospace;
        }
      }

      button {
        background: linear-gradient(135deg, $accent-blue, $accent-light);
        border: none;
        border-radius: 40px;
        padding: 12px 32px;
        font-weight: bold;
        font-size: 1rem;
        color: $bg-deep;
        cursor: pointer;
        box-shadow: 0 0 20px $accent-blue;
        transition: transform 0.2s, box-shadow 0.2s;

        &:hover:not(:disabled) {
          transform: scale(1.05);
          box-shadow: 0 0 30px $accent-light;
        }

        &:disabled {
          opacity: 0.5;
          cursor: not-allowed;
          box-shadow: none;
        }
      }
    }
  }

  /* 移动端适配 */
  @media (max-width: 768px) {
    padding-top: 90px;

    .carousel1 {
      display: none;
    }
    .carousel2 {
      display: block;
    }

    .board-header {
      left: 12px;
      transform: none;
      width: calc(100% - 24px);

      .title-wrap {
        flex-wrap: wrap;
        h1 {
          font-size: 1.4rem;
        }
        .subtitle {
          margin-left: 0;
        }
      }
    }

    .message-list {
      padding: 20px 12px 260px;
    }

    .message-card .message-content {
      padding-left: 0;
    }

    .message-form {
      left: 12px;
      transform: none;
      width: calc(100% - 24px);
    }
  }

  .sr-only {
    position: absolute !important;
    width: 1px !important;
    height: 1px !important;
    padding: 0 !important;
    margin: -1px !important;
    overflow: hidden !important;
    clip: rect(0, 0, 0, 0) !important;
    border: 0 !important;
  }
}
</style>
