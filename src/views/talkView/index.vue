<template>
  <div class="morning-chat">
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

    <!-- 静态机械纹理 -->
    <div class="hex-bg"></div>
    <div class="grid-bg"></div>

    <div class="chat-container">
      <!-- 统计面板（玻璃风格） -->
      <div class="stats-panel">
        <div class="stat-item">
          总对话次数：<span>{{ stats.totalChats }}</span>
        </div>
        <div class="stat-item">
          首次使用：<span>{{
            new Date(stats.firstTimestamp).toISOString().slice(0, 10)
          }}</span>
        </div>
        <div class="stat-item">
          活跃天数：<span>{{ stats.activeDates.length }}</span> 天
        </div>
        <div class="stat-item">
          今日对话：<span>{{
            stats.dailyChats[new Date().toISOString().slice(0, 10)] || 0
          }}</span>
          次
        </div>
        <button class="detail-btn" @click="showModal = true">全部</button>
      </div>

      <!-- 消息列表 -->
      <div class="messages" ref="msgList">
        <transition-group name="msg" tag="div">
          <div
            v-for="msg in chatLog"
            :key="msg.id"
            :class="[
              'message',
              msg.role,
              { error: msg.isError, egg: msg.isEgg },
            ]"
          >
            <div class="avatar" :class="msg.role"></div>
            <div class="bubble">
              <div class="content" v-html="msg.text"></div>
            </div>
          </div>
          <div v-if="loading" class="message bot" key="loading">
            <div class="avatar bot"></div>
            <div class="bubble loading">
              正在思考中
              <span class="dots">
                <span class="dot"></span>
                <span class="dot"></span>
                <span class="dot"></span>
              </span>
            </div>
          </div>
        </transition-group>
      </div>

      <!-- 输入区 -->
      <form class="input-area" @submit.prevent="sendMessage">
        <textarea
          v-model="input"
          placeholder="向莫宁提问…"
          :disabled="loading"
          @keydown="handleKeydown"
          rows="1"
        ></textarea>

        <div class="btn-group">
          <button
            type="button"
            class="clear-btn"
            @click="clearChat"
            :disabled="loading"
            title="清空对话"
          >
            ✖
          </button>
        </div>
        <button
          type="button"
          class="copy-btn"
          :disabled="!chatLog.length || loading"
          @click="() => copyChat()"
        >
          {{ copyButtonText }}
        </button>
        <!-- 发送按钮 -->
        <button
          type="submit"
          class="send-btn"
          :disabled="!input.trim() || loading"
        >
          发送
        </button>

        <!-- 统计数据按钮（移动端显示） -->
        <button
          type="button"
          class="Alldetail-btn"
          @click="showModal = true"
          title="查看统计"
        >
          统计数据
        </button>
      </form>
    </div>

    <!-- 详细统计弹窗 -->
    <div v-if="showModal" class="modal-overlay" @click.self="showModal = false">
      <div class="modal-content">
        <h3>演算统计</h3>
        <ul class="detail-list">
          <li>总对话次数：{{ stats.totalChats }}</li>
          <li>
            首次使用：{{
              new Date(stats.firstTimestamp).toISOString().slice(0, 10)
            }}
          </li>
          <li>活跃天数：{{ stats.activeDates.length }} 天</li>
          <li>
            今日对话：{{
              stats.dailyChats[new Date().toISOString().slice(0, 10)] || 0
            }}
            次
          </li>
          <li>总使用时长：{{ formatDuration(stats.totalTime) }}</li>
          <li>当前连续活跃：{{ stats.currentStreak }} 天</li>
          <li>最长连续活跃：{{ stats.longestStreak }} 天</li>
          <li>
            最活跃日：{{ mostActiveDayComputed }} （{{
              stats.dailyChats[mostActiveDayComputed] || 0
            }}
            次）
          </li>
        </ul>
        <button class="close-btn" @click="showModal = false">关闭</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
// 以下代码完全保留原有逻辑，未作任何修改
import {
  reactive,
  ref,
  computed,
  onMounted,
  nextTick,
  watch,
  onBeforeUnmount,
} from "vue";
import { sendMessageToHui } from "@/api/deepseekApi";

const STORAGE_KEY = "hui_chat_log";
const STORAGE_STATS_KEY = "hui_chat_stats";
const showModal = ref(false);

interface Stats {
  firstTimestamp: number;
  totalChats: number;
  activeDates: string[];
  dailyChats: Record<string, number>;
  currentStreak: number;
  longestStreak: number;
  totalTime: number;
}

const defaultStats: Stats = {
  firstTimestamp: Date.now(),
  totalChats: 0,
  activeDates: [],
  dailyChats: {},
  currentStreak: 0,
  longestStreak: 0,
  totalTime: 0,
};

function loadStats(): Stats {
  const saved = localStorage.getItem(STORAGE_STATS_KEY);
  if (saved) {
    try {
      const parsed = JSON.parse(saved);
      return { ...defaultStats, ...parsed };
    } catch {
      console.warn("加载统计数据失败，使用默认值");
    }
  }
  return { ...defaultStats };
}

function saveStats() {
  localStorage.setItem(STORAGE_STATS_KEY, JSON.stringify(stats));
}

function updateActive(date: string) {
  if (!stats.activeDates.includes(date)) {
    stats.activeDates.push(date);
    updateStreak();
    saveStats();
  }
}
function updateStreak() {
  const dates = [...stats.activeDates].sort();
  let curr = 0,
    max = stats.longestStreak,
    prevTs = 0;
  const todayStr = new Date().toISOString().slice(0, 10);
  dates.forEach((d) => {
    const ts = new Date(d).getTime();
    if (prevTs && ts - prevTs === 86400000) curr++;
    else curr = 1;
    max = Math.max(max, curr);
    prevTs = ts;
  });
  stats.currentStreak = dates[dates.length - 1] === todayStr ? curr : 0;
  stats.longestStreak = max;
  saveStats();
}

function updateDaily(date: string) {
  stats.dailyChats[date] = (stats.dailyChats[date] || 0) + 1;
  saveStats();
}

const mostActiveDayComputed = computed(() => {
  let day = "",
    max = 0;
  for (const [d, c] of Object.entries(stats.dailyChats)) {
    if (c > max) {
      max = c;
      day = d;
    }
  }
  return day || new Date().toISOString().slice(0, 10);
});

function formatDuration(ms: number): string {
  const totalMin = Math.floor(ms / 60000);
  const h = Math.floor(totalMin / 60);
  const m = totalMin % 60;
  return h ? `${h} 小时 ${m} 分钟` : `${m} 分钟`;
}

const stats = reactive<Stats>(loadStats());
const sessionStart = Date.now();

interface ChatMsg {
  id: number;
  role: "user" | "bot";
  text: string;
  isError?: boolean;
  isEgg?: boolean;
}

const chatLog = ref<ChatMsg[]>(loadChatLog());
const input = ref("");
const loading = ref(false);
const msgList = ref<HTMLElement>();

async function sendMessage() {
  if (!input.value.trim()) return;
  if (stats.totalChats === 0 && !localStorage.getItem(STORAGE_STATS_KEY)) {
    stats.firstTimestamp = Date.now();
    saveStats();
  }
  const date = new Date().toISOString().slice(0, 10);
  stats.totalChats++;
  updateActive(date);
  updateDaily(date);
  saveStats();

  const userText = input.value;
  chatLog.value.push({
    id: Date.now(),
    role: "user",
    text: userText,
  });
  input.value = "";
  loading.value = true;

  try {
    const history = chatLog.value.filter((msg) => !msg.isEgg && !msg.isError);
    const botReply = await sendMessageToHui(userText, history);
    if (botReply == "error") {
      chatLog.value.push({
        id: Date.now() + 2,
        role: "bot",
        text: "API余额耗尽了，去b站提醒我充钱吧(或者清空再试试)",
        isError: true,
      });
    } else {
      chatLog.value.push({
        id: Date.now() + 1,
        role: "bot",
        text: botReply,
      });
    }
  } catch (e) {
    console.error(e);
  } finally {
    loading.value = false;
    await scrollToBottom();
  }
}

function handleKeydown(e: KeyboardEvent) {
  if (e.key === "Enter") sendMessage();
}

function clearChat() {
  if (confirm("确定要清空全部对话吗？")) {
    chatLog.value = [
      {
        id: Date.now(),
        role: "bot",
        text: "……要不要坐近一点？这样说话比较方便。",
      },
    ];
    localStorage.removeItem(STORAGE_KEY);
  }
}

function loadChatLog(): ChatMsg[] {
  const saved = localStorage.getItem(STORAGE_KEY);
  if (saved) {
    try {
      return JSON.parse(saved);
    } catch (e) {
      console.error("chatLog 解析失败：", e);
    }
  }
  return [
    {
      id: Date.now(),
      role: "bot",
      text: "……要不要坐近一点？这样说话比较方便。",
    },
  ];
}

async function scrollToBottom() {
  await nextTick();
  if (msgList.value) {
    msgList.value.scrollTop = msgList.value.scrollHeight;
  }
}

const copyButtonText = ref("复制");
let _copyTimer: number | null = null;

function stripHtml(html = ""): string {
  if (typeof document === "undefined") {
    return html.replace(/<br\s*\/?>/gi, "\n").replace(/<[^>]+>/g, "");
  }
  const div = document.createElement("div");
  const withBreaks = String(html).replace(/<br\s*\/?>/gi, "\n");
  div.innerHTML = withBreaks;
  return div.textContent || div.innerText || "";
}

function buildChatPlainText(): string {
  return chatLog.value
    .map((msg) => {
      const time = new Date(msg.id).toLocaleString();
      const who = msg.role === "user" ? "你" : "莫宁";
      const text = stripHtml(msg.text);
      return `[${time}] ${who}: ${text}`;
    })
    .join("\n\n");
}

function fallbackCopyText(text: string) {
  const textarea = document.createElement("textarea");
  textarea.value = text;
  textarea.style.position = "fixed";
  textarea.style.left = "-9999px";
  textarea.style.top = "0";
  document.body.appendChild(textarea);
  textarea.select();
  textarea.setSelectionRange(0, textarea.value.length);
  const ok = document.execCommand("copy");
  document.body.removeChild(textarea);
  if (!ok) throw new Error("execCommand copy failed");
}

async function copyChat() {
  const text = buildChatPlainText();
  if (!text) {
    copyButtonText.value = "无内容可复制";
    clearTimeout(_copyTimer as number);
    _copyTimer = window.setTimeout(() => (copyButtonText.value = "复制"), 1600);
    return;
  }

  try {
    if (navigator.clipboard && navigator.clipboard.writeText) {
      await navigator.clipboard.writeText(text);
    } else {
      fallbackCopyText(text);
    }
    copyButtonText.value = "已复制";
  } catch (err) {
    try {
      fallbackCopyText(text);
      copyButtonText.value = "已复制";
    } catch (e) {
      console.error("复制失败：", e);
      copyButtonText.value = "复制失败";
    }
  } finally {
    clearTimeout(_copyTimer as number);
    _copyTimer = window.setTimeout(() => (copyButtonText.value = "复制"), 1600);
  }
}

watch(
  chatLog,
  async () => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(chatLog.value));
    await scrollToBottom();
  },
  { deep: true }
);

function handleBeforeUnload() {
  stats.totalTime += Date.now() - sessionStart;
  saveStats();
}

// ========== 背景图片导入与轮播 ==========
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
let Imgtimer: number | undefined;

onMounted(() => {
  scrollToBottom();
  window.addEventListener("beforeunload", handleBeforeUnload);
  Imgtimer = window.setInterval(() => {
    currentIndex.value =
      (currentIndex.value + 1) % Math.max(1, randomFive.value.length);
  }, 5200);
});

onBeforeUnmount(() => {
  window.removeEventListener("beforeunload", handleBeforeUnload);
  if (Imgtimer) clearInterval(Imgtimer);
  if (_copyTimer) clearTimeout(_copyTimer);
});
</script>

<style scoped lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;800&family=Rajdhani:wght@300;500;600&family=Share+Tech+Mono&display=swap");

/* 莫宁配色：深空机械 / 冰蓝光效 / 红瞳点缀（仅心形和错误） */
$bg-deep: #0a0c16; // 深空底色
$bg-panel: #121826; // 面板深色
$accent-blue: #4aa5ff; // 冰蓝主色
$accent-light: #9ac9ff; // 浅蓝高光
$red-pupil: #ff6b8b; // 红瞳（用于错误提示和心形）
$text-primary: #f0f6ff; // 冷白文字
$text-muted: rgba(240, 246, 255, 0.6);

.morning-chat {
  padding-top: 64px;
  min-height: 100vh;
  font-family: "Rajdhani", "Inter", "PingFang SC", sans-serif;
  color: $text-primary;
  display: flex;
  flex-direction: column;
  background: radial-gradient(ellipse at 30% 40%, #0e1424, $bg-deep);
  position: relative;
  overflow-x: hidden;

  /* 背景轮播（冷色滤镜） */
  .carousel {
    position: absolute;
    inset: 0;
    z-index: 0;
    pointer-events: none;

    &::after {
      content: "";
      position: absolute;
      inset: 0;
      background: rgba(10, 12, 22, 0.35);
    
      z-index: 1;
    }

    .carousel-image {
      position: absolute;
      width: 100%;
      height: 100%;
      object-fit: cover;
      opacity: 0;
      transition: opacity 1.2s ease;
   
      &.active {
        opacity: 1;
      }
    }
  }
  .carousel2 {
    display: none;
  }

  /* 静态机械纹理 */
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
    pointer-events: none;
    z-index: 1;
  }

  .grid-bg {
    position: absolute;
    inset: 0;
    background-image: linear-gradient(
        rgba(74, 165, 255, 0.02) 1px,
        transparent 1px
      ),
      linear-gradient(90deg, rgba(74, 165, 255, 0.02) 1px, transparent 1px);
    background-size: 40px 40px;
    pointer-events: none;
    z-index: 1;
  }

  /* 聊天主容器 */
  .chat-container {
    position: relative;
    z-index: 10;
    flex: 1;
    width: 920px;
    max-width: calc(100% - 32px);
    margin: 0 auto;
    padding: 24px;
    display: flex;
    flex-direction: column;
    gap: 16px;
  }

  /* 统计面板（玻璃风格） */
  .stats-panel {
    display: flex;
    align-items: center;
    background: rgba(10, 15, 30, 0.5);
    backdrop-filter: blur(12px);
    padding: 12px 20px;
    border-radius: 40px;
    border: 1px solid rgba(74, 165, 255, 0.2);
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
    font-size: 0.95rem;
    color: $text-primary;
    justify-content: space-around;
    margin-bottom: 10px;

    .stat-item {
      span {
        color: $accent-light;
        font-weight: 600;
        margin-left: 4px;
      }
    }

    .detail-btn {
      background: transparent;
      border: 1px solid rgba(74, 165, 255, 0.3);
      border-radius: 30px;
      padding: 6px 16px;
      color: $accent-light;
      cursor: pointer;
      font-size: 0.9rem;
      transition: all 0.2s;

      &:hover {
        background: rgba(74, 165, 255, 0.1);
        box-shadow: 0 0 15px $accent-blue;
      }
    }
  }

  /* 消息列表 */
  .messages {
    flex: 1;
    overflow-y: auto;
    padding: 12px 0 20px;
    overscroll-behavior: contain;
    scroll-behavior: smooth;
    height: 75vh;

    &::-webkit-scrollbar {
      width: 6px;
    }
    &::-webkit-scrollbar-thumb {
      background: $accent-blue;
      border-radius: 10px;
    }
  }

  /* 消息气泡 */
  .message {
    display: flex;
    align-items: flex-start;
    margin-bottom: 16px;

    &.user {
      flex-direction: row-reverse;
    }

    &.error .bubble {
      background: rgba(255, 107, 139, 0.1);
      border-color: $red-pupil;
    }

    .avatar {
      width: 48px;
      height: 48px;
      border-radius: 50%;
      margin: 0 12px;
      flex-shrink: 0;
      border: 2px solid rgba(74, 165, 255, 0.3);
      box-shadow: 0 0 20px $accent-blue;

      &.bot {
        background: linear-gradient(135deg, $accent-blue, $accent-light);
        background-image: url("@/assets/avatar/changli.png"); /* 可替换为莫宁头像 */
        background-size: cover;
        background-position: center;
     
      }

      &.user {
        background: $bg-panel;
        border: 2px solid $accent-light;
        position: relative;
  
      }
    }

    .bubble {
      max-width: 70%;
      background: rgba(10, 15, 30, 0.6);
      backdrop-filter: blur(8px);
      border: 1px solid rgba(74, 165, 255, 0.3);
      border-radius: 24px;
      padding: 12px 18px;
      color: $text-primary;
      line-height: 1.6;
      word-break: break-word;
      box-shadow: 0 10px 20px rgba(0, 0, 0, 0.3);
      transition: transform 0.2s, box-shadow 0.2s;

      &:hover {
        transform: translateY(-2px);
        box-shadow: 0 20px 30px rgba(0, 0, 0, 0.5);
      }

      &.loading {
        display: flex;
        align-items: center;
        gap: 8px;
      }

      .dots {
        display: inline-flex;
        gap: 4px;

        .dot {
          width: 6px;
          height: 6px;
          background: $accent-light;
          border-radius: 50%;
          animation: dotPulse 1s infinite;

          &:nth-child(2) {
            animation-delay: 0.2s;
          }
          &:nth-child(3) {
            animation-delay: 0.4s;
          }
        }
      }
    }

    &.user .bubble {
      background: rgba(74, 165, 255, 0.15);
      border-color: $accent-light;
    }
  }

  @keyframes dotPulse {
    0%,
    100% {
      opacity: 0.3;
      transform: scale(1);
    }
    50% {
      opacity: 1;
      transform: scale(1.2);
    }
  }

  /* 输入区 */
  .input-area {
 
    display: flex;
    align-items: center;
    gap: 12px;
    background: rgba(10, 15, 30, 0.7);
    backdrop-filter: blur(16px);
    border: 1px solid rgba(74, 165, 255, 0.3);
    border-radius: 60px;
    padding: 8px 8px 8px 20px;
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.6);

    textarea {
      flex: 1;
      background: transparent;
      border: none;
      color: $text-primary;
      font-size: 1rem;
      font-family: "Rajdhani", sans-serif;
      resize: none;
      outline: none;
      max-height: 120px;
      line-height: 1.5;

      &::placeholder {
        color: rgba(240, 246, 255, 0.3);
      }
    }

    .btn-group {
      display: flex;
      gap: 8px;

      .clear-btn {
        width: 40px;
        height: 40px;
        border-radius: 50%;
        background: rgba(0, 0, 0, 0.3);
        border: 1px solid rgba(74, 165, 255, 0.3);
        color: $text-primary;
        cursor: pointer;
        transition: all 0.2s;

        &:hover {
          background: rgba(74, 165, 255, 0.2);
          border-color: $accent-light;
        }
        &:disabled {
          opacity: 0.3;
          cursor: not-allowed;
        }
      }
    }

    .copy-btn {
      height: 40px;
      padding: 0 20px;
      border-radius: 40px;
      background: transparent;
      border: 1px solid rgba(74, 165, 255, 0.3);
      color: $accent-light;
      cursor: pointer;
      transition: all 0.2s;

      &:hover:not(:disabled) {
        background: rgba(74, 165, 255, 0.1);
        border-color: $accent-light;
      }
      &:disabled {
        opacity: 0.3;
        cursor: not-allowed;
      }
    }

    .send-btn {
      height: 40px;
      padding: 0 28px;
      border: none;
      border-radius: 40px;
      background: linear-gradient(135deg, $accent-blue, $accent-light);
      color: $bg-deep;
      font-weight: 600;
      cursor: pointer;
      box-shadow: 0 0 20px $accent-blue;
      transition: transform 0.2s, box-shadow 0.2s;

      &:hover:not(:disabled) {
        transform: scale(1.05);
        box-shadow: 0 0 30px $accent-light;
      }
      &:disabled {
        opacity: 0.3;
        cursor: not-allowed;
      }
    }

    .Alldetail-btn {
      display: none; /* 默认隐藏，移动端显示 */
    }
  }

  /* 弹窗 */
  .modal-overlay {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.8);
    backdrop-filter: blur(8px);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 2000;

    .modal-content {
      width: 400px;
      max-width: 90%;
      background: $bg-panel;
      border: 1px solid rgba(74, 165, 255, 0.3);
      border-radius: 30px;
      padding: 30px;
      box-shadow: 0 30px 60px rgba(0, 0, 0, 0.8);

      h3 {
        margin: 0 0 20px;
        font-family: "Orbitron", sans-serif;
        font-size: 1.5rem;
        color: $accent-light;
        text-align: center;
      }

      .detail-list {
        list-style: none;
        padding: 0;
        margin: 0 0 30px;
        font-size: 1rem;
        line-height: 2;

        li {
          border-bottom: 1px solid rgba(74, 165, 255, 0.1);
          padding: 8px 0;
        }
      }

      .close-btn {
        display: block;
        width: 100%;
        padding: 12px;
        border: none;
        border-radius: 40px;
        background: linear-gradient(135deg, $accent-blue, $accent-light);
        color: $bg-deep;
        font-weight: 600;
        cursor: pointer;
        transition: transform 0.2s;

        &:hover {
          transform: scale(1.02);
        }
      }
    }
  }

  /* 移动端响应式 */
  @media (max-width: 768px) {
    .carousel1 {
      display: none;
    }
    .carousel2 {
      display: block;
    }

    .chat-container {
      width: 100%;
      padding: 12px;
    }

    .stats-panel {
      display: none; /* 移动端隐藏统计条，用底部按钮代替 */
    }

    .input-area {
      flex-wrap: wrap;
      border-radius: 30px;
      padding: 12px;

      textarea {
        width: 100%;
        margin-bottom: 8px;
      }

      .btn-group {
        order: 2;
      }

      .copy-btn {
        order: 3;
      }

      .send-btn {
        order: 4;
      }

      .Alldetail-btn {
        display: block; /* 移动端显示统计按钮 */
        order: 1;
        width: 100%;
        margin-bottom: 8px;
        height: 40px;
        border: 1px solid rgba(74, 165, 255, 0.3);
        background: transparent;
        color: $accent-light;
        border-radius: 40px;
        cursor: pointer;

        &:hover {
          background: rgba(74, 165, 255, 0.1);
        }
      }
    }

    .message .avatar {
      width: 36px;
      height: 36px;
      margin: 0 8px;
    }

    .message .bubble {
      max-width: 80%;
      font-size: 0.9rem;
    }
  }
}
</style>
