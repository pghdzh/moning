<template>
  <section
    class="morning-player"
    @keydown.space.prevent="onSpace"
    tabindex="0"
    ref="rootEl"
    aria-label="莫宁 音乐播放器"
  >
    <!-- 背景机械纹理（静态） -->
    <div class="grid-bg"></div>
    <div class="hex-bg"></div>

    <!-- 动态装饰 -->
    <div class="scan-line"></div>
    <div class="data-flow"></div>
    <div class="red-dot red-dot-1"></div>
    <div class="red-dot red-dot-2"></div>
    <div class="red-dot red-dot-3"></div>
    <div class="red-dot red-dot-4"></div>
    <div class="mech-corner"></div>
    <div class="floating-particles">
      <span v-for="n in 6" :key="n" class="particle"></span>
    </div>

    <div class="stage">
      <!-- 左侧：封面与控制 -->
      <div class="left" role="region" aria-label="播放器控制区">
        <div class="cover" :style="coverStyle">
          <video
            v-if="videoSrc"
            class="video-background"
            :src="videoSrc"
            autoplay
            muted
            loop
            playsinline
            aria-hidden="true"
            tabindex="-1"
            :class="videoClass"
          ></video>
          <!-- 冷色滤镜覆盖层 -->
          <div class="cover-overlay"></div>

          <!-- 加载遮罩 -->
          <div v-if="loadingAudio" class="loading-overlay" aria-hidden="true">
            <div class="spinner" />
            <div class="loading-text">演算中…</div>
          </div>
        </div>

        <div class="controls">
          <div class="title" :title="current?.title || '未选择曲目'">
            {{ current?.title || "未选择曲目" }}
          </div>

          <div class="meta">
            <span class="time">{{ formatTime(currentTime) }}</span>
            <span class="divider">/</span>
            <span class="time">{{ formatTime(duration) }}</span>
          </div>

          <div
            class="progress-wrap"
            ref="progressWrap"
            @click="seekByClick"
            @pointerdown.prevent="onPointerDownProgress"
            role="slider"
            :aria-valuemin="0"
            :aria-valuemax="duration"
            :aria-valuenow="currentTime"
            aria-label="进度条"
          >
            <div class="progress-bar">
              <div
                class="progress"
                :style="{ width: progressPercent + '%' }"
              ></div>
            </div>
            <div
              class="progress-handle"
              :style="{ left: progressPercent + '%' }"
              aria-hidden="true"
            ></div>
          </div>

          <div class="btns">
            <button class="icon" @click="prev" aria-label="上一首">⏴</button>

            <button
              class="play"
              @click="togglePlay"
              :aria-pressed="playing"
              :aria-label="playing ? '暂停' : '播放'"
            >
              <span v-if="!playing">▶</span>
              <span v-else>❚❚</span>
            </button>

            <button class="icon" @click="next" aria-label="下一首">⏵</button>

            <div class="modes" role="group" aria-label="播放模式">
              <button
                :class="{ active: shuffle }"
                @click="toggleShuffle"
                title="随机播放"
              >
                🔀
              </button>
              <button
                :class="{ active: repeatMode !== 'off' }"
                @click="toggleRepeat"
                title="循环模式"
              >
                🔁
              </button>
            </div>

            <div class="volume" aria-label="音量控制">
              <input
                type="range"
                min="0"
                max="1"
                step="0.01"
                v-model.number="volume"
                aria-label="音量"
              />
            </div>
          </div>

          <div v-if="errorMessage" class="error-msg" role="status">
            {{ errorMessage }}
          </div>
        </div>
      </div>

      <!-- 右侧：播放列表 -->
      <div class="right" role="region" aria-label="播放列表">
        <div class="playlist-header">
          <div class="left-head">
            <h3>演算列表</h3>
            <div class="api-hint">
              {{ loading ? "加载中…" : list.length ? "" : "目录为空" }}
            </div>
          </div>

          <div class="search-wrap">
            <input
              v-model="searchTerm"
              @input="onSearchInput"
              placeholder="搜索曲名..."
              aria-label="搜索曲目"
            />
            <button
              v-if="searchTerm"
              class="clear"
              @click="clearSearch"
              aria-label="清除搜索"
            >
              ✕
            </button>
          </div>
        </div>

        <div class="list-area">
          <div v-if="loading" class="list-loading">
            <div class="small-spinner" />
            加载目录...
          </div>

          <ul class="playlist" role="list">
            <li
              v-for="(item, idx) in filteredList"
              :key="item.name || idx"
              :class="{ active: idx === index }"
              @click="selectTrack(idx)"
              tabindex="0"
              @keyup.enter="selectTrack(idx)"
              role="listitem"
              :aria-current="idx === index ? 'true' : 'false'"
            >
              <div class="left-col">
                <div class="dot" aria-hidden="true"></div>
                <div class="title" :title="item.title">{{ item.title }}</div>
              </div>
              <div class="right-col">
                <div class="len">
                  {{ item.duration ? formatTime(item.duration) : "--:--" }}
                </div>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>

    <audio
      ref="audioRef"
      @timeupdate="onTimeUpdate"
      @ended="onEnded"
      @loadedmetadata="onLoadedMetadata"
      @error="onAudioError"
      preload="metadata"
    ></audio>
  </section>
</template>

<script setup lang="ts">
import {
  ref,
  computed,
  onMounted,
  onBeforeUnmount,
  watch,
  nextTick,
} from "vue";
import { getMusicList, getMusicUrl } from "@/api/modules/music";

type MusicItem = {
  name: string;
  title: string;
  url?: string;
  duration?: number | null;
};

const list = ref<MusicItem[]>([]);
const loading = ref(false);
const index = ref<number>(-1);
const playing = ref(false);
const audioRef = ref<HTMLAudioElement | null>(null);
const currentTime = ref<number>(0);
const duration = ref<number>(0);
const volume = ref<number>(
  Number(localStorage.getItem("cantarella_volume") ?? 0.8)
);
const shuffle = ref<boolean>(false);
const repeatMode = ref<"off" | "one" | "all">("off");

const rootEl = ref<HTMLElement | null>(null);
const progressWrap = ref<HTMLElement | null>(null);
const dragging = ref(false);
const errorMessage = ref<string | null>(null);
const loadingAudio = ref(false);

const isMobile = ref<boolean>(window.innerWidth <= 920);
window.addEventListener("resize", () => {
  isMobile.value = window.innerWidth <= 920;
});

const videoSrc = ref("");
const videoClass = ref("");

const searchTerm = ref("");
let searchTimer: any = null;
const searchDebounceMs = 240;

const current = computed(() =>
  index.value >= 0 && list.value[index.value] ? list.value[index.value] : null
);
const progressPercent = computed(() =>
  duration.value
    ? Math.min(100, Math.max(0, (currentTime.value / duration.value) * 100))
    : 0
);

const coverStyle = computed(() => {
  const t = current.value?.title || "morning";
  let hash = 0;
  for (let i = 0; i < t.length; i++)
    hash = (hash << 5) - hash + t.charCodeAt(i);
  const r1 = (Math.abs(hash) % 60) + 20;
  const r2 = (Math.abs(hash * 3) % 60) + 20;
  return {
    background: `radial-gradient(circle at 28% 28%, rgba(74,165,255,0.12), transparent 12%), linear-gradient(135deg, rgba(${r1},${r2},255,0.1), rgba(10,15,30,0.3))`,
  };
});

const filteredList = computed(() => {
  const term = (searchTerm.value || "").trim().toLowerCase();
  if (!term) return list.value;
  return list.value.filter((i) => (i.title || "").toLowerCase().includes(term));
});

async function fetchList() {
  loading.value = true;
  try {
    const res = await getMusicList();
    const items =
      res?.ok && Array.isArray(res.list)
        ? res.list
        : Array.isArray(res)
        ? res
        : res?.list ?? [];
    list.value = items.map((it: any) => ({
      name: it.name,
      title: it.title ?? (it.name ? it.name.replace(/\.mp3$/i, "") : "未知"),
      url: getMusicUrl(it.name),
      duration: null,
    }));
  } catch (e) {
    console.error("获取音乐列表失败", e);
    list.value = [];
    errorMessage.value = "加载目录失败";
  } finally {
    loading.value = false;
  }
}

async function safeSetSrc(url: string) {
  const a = audioRef.value!;
  errorMessage.value = null;
  loadingAudio.value = true;
  try {
    try {
      const head = await fetch(url, { method: "HEAD" });
      if (!head.ok) throw new Error(`资源响应 ${head.status}`);
      const ct = head.headers.get("content-type") || "";
      if (!ct.includes("audio")) {
        console.warn("content-type 不是 audio:", ct);
      }
    } catch (e) {}
    a.src = url;
    a.load();
  } catch (err) {
    console.error("设置音源失败", err);
    errorMessage.value = "无法加载音频资源";
    throw err;
  }
}

async function loadCurrent(doPlay = false) {
  const a = audioRef.value;
  const curr = current.value;
  if (!a || !curr) return;
  a.pause();
  duration.value = 0;
  currentTime.value = 0;
  try {
    await safeSetSrc(curr.url || getMusicUrl(curr.name));
    if (doPlay) await play();
  } catch {
    playing.value = false;
    loadingAudio.value = false;
  }
}

async function play() {
  const a = audioRef.value;
  if (!a) return;
  try {
    await a.play();
    playing.value = true;
    errorMessage.value = null;
  } catch (e: any) {
    console.warn("播放失败", e);
    playing.value = false;
    errorMessage.value = "播放被浏览器阻止或资源不可用";
  }
}
function pause() {
  audioRef.value?.pause();
  playing.value = false;
}
function togglePlay() {
  if (!audioRef.value) return;
  if (playing.value) pause();
  else play();
}

function selectTrack(i: number) {
  if (i < 0 || i >= list.value.length) return;
  index.value = i;
  loadCurrent(true);
}

function onTimeUpdate(e: Event) {
  const t = e.target as HTMLAudioElement;
  currentTime.value = t.currentTime || 0;
}
function onLoadedMetadata(e: Event) {
  const t = e.target as HTMLAudioElement;
  duration.value = isFinite(t.duration) ? t.duration : 0;
  if (current.value && !current.value.duration)
    current.value.duration = duration.value;
  loadingAudio.value = false;
}
function onEnded() {
  loadingAudio.value = false;
  if (repeatMode.value === "one") {
    if (audioRef.value) {
      audioRef.value.currentTime = 0;
      play();
    }
    return;
  }
  if (shuffle.value) {
    playRandom();
    return;
  }
  if (index.value < list.value.length - 1) selectTrack(index.value + 1);
  else {
    if (repeatMode.value === "all") selectTrack(0);
    else playing.value = false;
  }
}
function onAudioError(e: Event) {
  const a = audioRef.value;
  console.error("audio error", a?.error);
  errorMessage.value = "音频播放出错";
  playing.value = false;
  loadingAudio.value = false;
}

function next() {
  if (!list.value.length) return;
  if (shuffle.value) {
    playRandom();
    return;
  }
  if (index.value < list.value.length - 1) selectTrack(index.value + 1);
  else if (repeatMode.value === "all") selectTrack(0);
}
function prev() {
  if (!audioRef.value) return;
  if (audioRef.value.currentTime > 4) {
    audioRef.value.currentTime = 0;
    return;
  }
  if (index.value > 0) selectTrack(index.value - 1);
  else if (repeatMode.value === "all") selectTrack(list.value.length - 1);
}
function playRandom() {
  if (!list.value.length) return;
  if (list.value.length === 1) {
    selectTrack(0);
    return;
  }
  let i = index.value;
  while (i === index.value) i = Math.floor(Math.random() * list.value.length);
  selectTrack(i);
}

function seekByClick(e: MouseEvent | TouchEvent) {
  if (!progressWrap.value || !duration.value || !audioRef.value) return;
  const rect = progressWrap.value.getBoundingClientRect();
  const clientX =
    (e as MouseEvent).clientX ?? (e as TouchEvent).touches?.[0]?.clientX;
  if (clientX == null) return;
  const x = Math.min(Math.max(0, clientX - rect.left), rect.width);
  const ratio = x / rect.width;
  audioRef.value.currentTime = ratio * duration.value;
  currentTime.value = audioRef.value.currentTime;
}
function onPointerDownProgress(e: PointerEvent) {
  if (!progressWrap.value || !audioRef.value || !duration.value) return;
  dragging.value = true;
  (e.target as Element).setPointerCapture?.(e.pointerId);
  window.addEventListener("pointermove", onPointerMoveProgress);
  window.addEventListener("pointerup", onPointerUpProgress);
  handlePointer(e);
}
function onPointerMoveProgress(e: PointerEvent) {
  handlePointer(e);
}
function onPointerUpProgress(e: PointerEvent) {
  dragging.value = false;
  window.removeEventListener("pointermove", onPointerMoveProgress);
  window.removeEventListener("pointerup", onPointerUpProgress);
}
function handlePointer(e: PointerEvent) {
  if (!progressWrap.value || !audioRef.value || !duration.value) return;
  const rect = progressWrap.value.getBoundingClientRect();
  const x = Math.min(Math.max(0, e.clientX - rect.left), rect.width);
  const ratio = x / rect.width;
  audioRef.value.currentTime = ratio * duration.value;
  currentTime.value = audioRef.value.currentTime;
}

watch(volume, (v) => {
  if (audioRef.value) audioRef.value.volume = v;
  localStorage.setItem("cantarella_volume", String(v));
});

function toggleShuffle() {
  shuffle.value = !shuffle.value;
}
function toggleRepeat() {
  if (repeatMode.value === "off") repeatMode.value = "all";
  else if (repeatMode.value === "all") repeatMode.value = "one";
  else repeatMode.value = "off";
}

function onSpace() {
  if (
    document.activeElement &&
    ["INPUT", "TEXTAREA"].includes(document.activeElement.tagName)
  )
    return;
  togglePlay();
}

function onSearchInput() {
  if (searchTimer) clearTimeout(searchTimer);
  searchTimer = setTimeout(() => {
    searchTimer = null;
  }, searchDebounceMs);
}
function clearSearch() {
  searchTerm.value = "";
}

function formatTime(sec?: number) {
  if (!sec || !isFinite(sec)) return "--:--";
  const s = Math.floor(sec % 60);
  const m = Math.floor(sec / 60);
  return `${String(m).padStart(2, "0")}:${String(s).padStart(2, "0")}`;
}

onMounted(async () => {
  audioRef.value =
    (document.querySelector(".morning-player audio") as HTMLAudioElement) ??
    null;
  if (audioRef.value) audioRef.value.volume = volume.value;

  // 修正视频选择：PC端（非移动）使用横版/mp1，移动端使用竖版/mp2
  const folder = isMobile.value ? "/mp1" : "/mp2";
  const idx = Math.floor(Math.random() * 4) + 1;
  videoSrc.value = `${folder}/1 (${idx}).mp4`;
  videoClass.value = isMobile.value ? "portrait" : "landscape";

  await fetchList();

  window.addEventListener("keydown", globalKeydown);
});

onBeforeUnmount(() => {
  window.removeEventListener("keydown", globalKeydown);
});

function globalKeydown(e: KeyboardEvent) {
  if (e.code === "Space") {
    if (
      document.activeElement &&
      ["INPUT", "TEXTAREA"].includes(document.activeElement.tagName)
    )
      return;
    e.preventDefault();
    togglePlay();
  } else if (e.code === "Escape") {
    pause();
  }
}
</script>

<style scoped lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;800&family=Rajdhani:wght@300;500;600&family=Share+Tech+Mono&display=swap");

/* 莫宁配色 */
$bg-deep: #0a0c16;
$bg-panel: #121826;
$accent-blue: #4aa5ff;
$accent-light: #9ac9ff;
$red-pupil: #ff6b8b;
$text-primary: #f0f6ff;
$text-muted: rgba(240, 246, 255, 0.6);

.morning-player {
  position: relative;
  padding: 20px;
  min-height: 100vh;
  background: radial-gradient(ellipse at 30% 40%, #0e1424, $bg-deep);
  color: $text-primary;
  padding-top: 80px;
  font-family: "Rajdhani", "Inter", "PingFang SC", sans-serif;
  outline: none;
  overflow: hidden;

  /* 静态机械纹理 */
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
    z-index: 0;
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
    pointer-events: none;
    z-index: 0;
  }

  /* 扫描线 */
  .scan-line {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 2px;
    background: linear-gradient(
      90deg,
      transparent,
      rgba(74, 165, 255, 0.5) 20%,
      rgba(200, 230, 255, 0.7) 50%,
      rgba(74, 165, 255, 0.5) 80%,
      transparent
    );
    filter: blur(3px);
    animation: scanMove 6s linear infinite;
    z-index: 2;
    pointer-events: none;
    opacity: 0.5;
  }

  @keyframes scanMove {
    0% {
      top: 0;
      opacity: 0.3;
    }
    10% {
      opacity: 0.8;
    }
    90% {
      opacity: 0.8;
    }
    100% {
      top: 100%;
      opacity: 0.3;
    }
  }

  /* 数据流 */
  .data-flow {
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 4px;
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
    z-index: 2;
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

  /* 红瞳微光点缀 */
  .red-dot {
    position: absolute;
    width: 4px;
    height: 4px;
    background: $red-pupil;
    border-radius: 50%;
    filter: blur(2px);
    box-shadow: 0 0 12px $red-pupil;
    opacity: 0.5;
    animation: redBreath 3s infinite;
    z-index: 2;
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

  /* 浮动粒子 */
  .floating-particles {
    position: absolute;
    inset: 0;
    pointer-events: none;
    z-index: 1;
  }
  .particle {
    position: absolute;
    width: 2px;
    height: 2px;
    background: $accent-light;
    border-radius: 50%;
    box-shadow: 0 0 8px $accent-blue;
    opacity: 0.3;
    animation: floatParticle 10s infinite;
    &:nth-child(1) {
      top: 20%;
      left: 15%;
      animation-delay: 0s;
    }
    &:nth-child(2) {
      top: 70%;
      left: 85%;
      animation-delay: 2s;
    }
    &:nth-child(3) {
      top: 40%;
      left: 50%;
      animation-delay: 4s;
    }
    &:nth-child(4) {
      top: 80%;
      left: 30%;
      animation-delay: 1s;
    }
    &:nth-child(5) {
      top: 10%;
      left: 90%;
      animation-delay: 3s;
    }
    &:nth-child(6) {
      top: 60%;
      left: 10%;
      animation-delay: 5s;
    }
  }

  @keyframes floatParticle {
    0% {
      transform: translate(0, 0);
      opacity: 0.2;
    }
    50% {
      transform: translate(20px, -20px);
      opacity: 0.6;
    }
    100% {
      transform: translate(0, 0);
      opacity: 0.2;
    }
  }

  /* 主区域布局 */
  .stage {
    position: relative;
    z-index: 10;
    display: flex;
    gap: 18px;
    max-width: 1100px;
    margin: 0 auto;
    align-items: flex-start;
  }

  /* 左侧 */
  .left {
    width: 420px;
    background: rgba(10, 15, 30, 0.5);
    backdrop-filter: blur(12px);
    border: 1px solid rgba(74, 165, 255, 0.2);
    border-radius: 24px;
    padding: 18px;
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.6);
    position: relative;
  }

  /* cover 区 */
  .cover {
    width: 100%;
    height: 620px;
    border-radius: 16px;
    overflow: hidden;
    position: relative;
    box-shadow: 0 0 30px rgba(74, 165, 255, 0.3);
    border: 1px solid rgba(74, 165, 255, 0.3);
    display: flex;
    align-items: center;
    justify-content: center;

    .video-background {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }
    /* 横版视频（PC） */
    .landscape {
      object-fit: cover; /* 保持铺满，可能裁剪，但 cover 效果最佳 */
    }
    /* 竖版视频（移动） */
    .portrait {
      object-fit: cover;
      width: auto;
      height: 100%;
    }
    .cover-overlay {
      position: absolute;
      inset: 0;
      background: linear-gradient(
        135deg,
        rgba(74, 165, 255, 0.1),
        rgba(10, 15, 30, 0.5)
      );
      pointer-events: none;
    }
  }

  /* 加载遮罩 */
  .loading-overlay {
    position: absolute;
    inset: 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background: rgba(0, 0, 0, 0.6);
    backdrop-filter: blur(4px);
    z-index: 8;
  }
  .spinner,
  .small-spinner {
    width: 40px;
    height: 40px;
    border-radius: 50%;
    border: 4px solid rgba(74, 165, 255, 0.2);
    border-top-color: $accent-blue;
    animation: spin 1s linear infinite;
  }
  .small-spinner {
    width: 20px;
    height: 20px;
    border-width: 3px;
  }
  .loading-text {
    margin-top: 8px;
    color: $accent-light;
    font-family: "Share Tech Mono", monospace;
  }

  @keyframes spin {
    to {
      transform: rotate(360deg);
    }
  }

  /* 控件 */
  .controls {
    margin-top: 16px;
  }
  .title {
    font-size: 1.2rem;
    font-weight: 600;
    color: $accent-light;
    letter-spacing: 1px;
    font-family: "Orbitron", sans-serif;
    text-shadow: 0 0 10px $accent-blue;
  }
  .meta {
    margin-top: 8px;
    font-size: 0.9rem;
    color: $text-muted;
    display: flex;
    gap: 8px;
    align-items: center;
  }

  /* 进度条 */
  .progress-wrap {
    margin-top: 16px;
    cursor: pointer;
    touch-action: none;
    position: relative;
  }
  .progress-bar {
    height: 8px;
    background: rgba(255, 255, 255, 0.1);
    border-radius: 10px;
    overflow: hidden;
    position: relative;
  }
  .progress {
    height: 100%;
    width: 0%;
    background: linear-gradient(90deg, $accent-blue, $accent-light);
    box-shadow: 0 0 10px $accent-blue;
    transition: width 0.1s linear;
  }
  .progress-handle {
    position: absolute;
    top: -4px;
    width: 16px;
    height: 16px;
    border-radius: 50%;
    background: $accent-light;
    transform: translateX(-50%);
    box-shadow: 0 0 20px $accent-blue;
    border: 2px solid $bg-deep;
  }

  /* 按钮行 */
  .btns {
    margin-top: 16px;
    display: flex;
    align-items: center;
    gap: 12px;
    flex-wrap: wrap;
  }
  .icon,
  .play {
    border: none;
    background: transparent;
    color: $text-primary;
    font-size: 1.2rem;
    cursor: pointer;
    padding: 8px 12px;
    border-radius: 30px;
    transition: transform 0.2s, background 0.2s, box-shadow 0.2s;
    border: 1px solid rgba(74, 165, 255, 0.3);

    &:hover {
      transform: translateY(-3px);
      background: rgba(74, 165, 255, 0.1);
      box-shadow: 0 0 20px $accent-blue;
    }
  }
  .play {
    background: linear-gradient(135deg, $accent-blue, $accent-light);
    color: $bg-deep;
    font-weight: 600;
    border: none;
    padding: 8px 20px;
  }
  .modes button {
    background: transparent;
    border: 1px solid rgba(74, 165, 255, 0.3);
    color: $text-primary;
    padding: 6px 12px;
    border-radius: 30px;
    margin-right: 6px;
    cursor: pointer;
    transition: all 0.2s;

    &.active {
      background: rgba(74, 165, 255, 0.2);
      border-color: $accent-blue;
      box-shadow: 0 0 15px $accent-blue;
    }
  }
  .volume input[type="range"] {
    width: 100px;
    height: 6px;
    -webkit-appearance: none;
    background: rgba(255, 255, 255, 0.1);
    border-radius: 10px;
    outline: none;

    &::-webkit-slider-thumb {
      -webkit-appearance: none;
      width: 16px;
      height: 16px;
      border-radius: 50%;
      background: $accent-light;
      box-shadow: 0 0 10px $accent-blue;
      cursor: pointer;
    }
  }

  /* 错误信息 */
  .error-msg {
    margin-top: 10px;
    color: $red-pupil;
    font-weight: 500;
  }

  /* 右侧列表 */
  .right {
    flex: 1;
    max-height: 68vh;
    overflow: hidden;
    border-radius: 24px;
    background: rgba(10, 15, 30, 0.5);
    backdrop-filter: blur(12px);
    border: 1px solid rgba(74, 165, 255, 0.2);
    padding: 16px;
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.6);
    display: flex;
    flex-direction: column;
    transition: max-height 0.3s ease;
  }

  .playlist-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 12px;
    gap: 12px;

    .left-head {
      display: flex;
      gap: 12px;
      align-items: center;

      h3 {
        margin: 0;
        font-family: "Orbitron", sans-serif;
        color: $accent-light;
        font-size: 1.2rem;
      }

      .api-hint {
        color: $text-muted;
        font-size: 0.9rem;
      }
    }

    .search-wrap {
      display: flex;
      align-items: center;
      gap: 8px;

      input {
        padding: 8px 16px;
        border-radius: 30px;
        border: 1px solid rgba(74, 165, 255, 0.3);
        background: rgba(0, 0, 0, 0.3);
        color: $text-primary;
        font-size: 0.9rem;
        outline: none;
        width: 200px;

        &:focus {
          border-color: $accent-blue;
          box-shadow: 0 0 15px $accent-blue;
        }
      }

      .clear {
        background: transparent;
        border: none;
        color: $text-muted;
        cursor: pointer;
        font-size: 1rem;

        &:hover {
          color: $accent-light;
        }
      }
    }
  }

  .list-area {
    flex: 1;
    overflow-y: auto;
    padding-right: 4px;

    &::-webkit-scrollbar {
      width: 4px;
    }
    &::-webkit-scrollbar-thumb {
      background: $accent-blue;
      border-radius: 10px;
    }
  }

  .list-loading {
    display: flex;
    align-items: center;
    gap: 8px;
    color: $text-muted;
    padding: 20px;
  }

  .playlist {
    list-style: none;
    padding: 0;
    margin: 0;

    li {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 10px 12px;
      border-radius: 16px;
      cursor: pointer;
      transition: background 0.2s, transform 0.2s;
      border: 1px solid transparent;
      margin-bottom: 4px;

      &:hover {
        background: rgba(74, 165, 255, 0.1);
        transform: translateX(4px);
        border-color: rgba(74, 165, 255, 0.3);
      }

      &.active {
        background: rgba(74, 165, 255, 0.2);
        border-color: $accent-blue;
        box-shadow: 0 0 15px $accent-blue;
      }

      .left-col {
        display: flex;
        align-items: center;
        gap: 12px;
        min-width: 0;
        flex: 1;

        .dot {
          width: 8px;
          height: 8px;
          border-radius: 50%;
          background: $accent-light;
          box-shadow: 0 0 10px $accent-blue;
        }

        .title {
          font-size: 0.95rem;
          color: $text-primary;
          white-space: nowrap;
          overflow: hidden;
          text-overflow: ellipsis;
        }
      }

      .right-col .len {
        color: $text-muted;
        font-size: 0.85rem;
        font-family: "Share Tech Mono", monospace;
      }
    }
  }

  /* 响应式 */
  @media (max-width: 920px) {
    .stage {
      flex-direction: column;
    }
    .left {
      width: 100%;
    }
    /* 移动端封面高度适当减小，适应竖屏 */
    .cover {
      height: 400px; /* 移动端更矮，使整体比例协调 */
    }
    .right {
      width: 100%;
      max-height: none;
    }

    .playlist-header {
      flex-direction: column;
      align-items: flex-start;
    }

    .search-wrap input {
      width: 150px;
    }
  }

  @media (max-width: 520px) {
    .btns {
      justify-content: center;
    }
    .volume input {
      width: 80px;
    }
    .cover {
      height: 300px; /* 小屏进一步降低 */
    }
  }
}
</style>
