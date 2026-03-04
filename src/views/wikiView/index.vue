<template>
  <div class="morning-wiki">
    <!-- 背景轮播（冷色滤镜） -->
    <div class="carousel">
      <img
        v-for="(src, idx) in randomFive"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>

    <!-- 静态机械纹理 -->
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

    <header class="wiki-header">
      <div class="title">
        <h1>演构文本库</h1>
        <p class="subtitle">跨越时空的鸿沟，与你共享每一页星光</p>
      </div>
      <div class="actions">
        <input
          v-model="search"
          class="search"
          placeholder="搜索标题或者标签..."
        />
        <button class="btn btn-new" @click="openCreate">新建词条</button>
      </div>
    </header>

    <main class="wiki-body">
      <div v-if="filteredEntries.length === 0" class="empty">
        没有找到匹配的词条 ✨
      </div>

      <ul class="entry-list">
        <li v-for="entry in filteredEntries" :key="entry.id" class="entry-card">
          <div class="entry-head">
            <div class="entry-meta" @click="openDetail(entry)">
              <div class="entry-title-wrap">
                <h2 class="entry-title">{{ entry.title }}</h2>
                <span class="entry-badge">#{{ entry.slug || "未设置" }}</span>
              </div>
              <div class="entry-info">
                <span class="meta-item">作者：{{ entry.author }}</span>
                <span class="meta-item"
                  >时间：{{ formatTime(entry.updatedAt) }}</span
                >
              </div>
            </div>

            <div class="entry-actions">
              <button
                class="like"
                :class="{ active: isLiked(entry.id) }"
                :aria-pressed="isLiked(entry.id)"
                @click.stop="toggleLike(entry.id)"
              >
                <img
                  :src="
                    isLiked(entry.id)
                      ? '/icons/heart-red-filled.svg'
                      : '/icons/heart-red-outline.svg'
                  "
                  alt="like"
                />
                <span class="like-count">{{ entry.likes }}</span>
              </button>
              <div class="edit-delete" v-if="canEdit(entry.id)">
                <button class="small" @click="openEdit(entry)">编辑</button>
                <button class="small danger" @click="remove(entry.id)">
                  删除
                </button>
              </div>
            </div>
          </div>
        </li>
      </ul>
    </main>

    <!-- Edit/Create Modal -->
    <transition name="fade-zoom">
      <div class="modal-overlay" v-if="showModal">
        <div class="modal">
          <header class="modal-header">
            <h3>{{ editing ? "编辑词条" : "新建词条" }}</h3>
            <button class="close" @click="closeModal">✕</button>
          </header>
          <section class="modal-body">
            <label>
              标题
              <input v-model="form.title" placeholder="输入标题" />
            </label>

            <label>
              词条（短标签）
              <input
                v-model="form.slug"
                placeholder="比如：彩蛋、考据、点位等等"
              />
            </label>

            <label>
              作者
              <input v-model="form.author" placeholder="作者昵称" />
            </label>

            <label>
              内容
              <textarea
                v-model="form.content"
                rows="8"
                placeholder="在这里输入词条内容"
              ></textarea>
            </label>
          </section>
          <footer class="modal-footer">
            <button class="btn ghost" @click="closeModal">取消</button>
            <button class="btn" @click="submit">
              {{ editing ? "保存" : "创建" }}
            </button>
          </footer>
        </div>
      </div>
    </transition>

    <!-- Detail Modal -->
    <transition name="fade-zoom">
      <div class="modal-overlay" v-if="detailEntry">
        <div class="modal detail-modal">
          <header class="modal-header">
            <h3>{{ detailEntry.title }}</h3>
            <button class="close" @click="detailEntry = null">✕</button>
          </header>
          <section class="modal-body">
            <div class="detail-content">{{ detailEntry.content }}</div>
          </section>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup lang="ts">
// 以下 script 部分完全保留原有逻辑，未作任何修改
import { ref, reactive, computed, onMounted, onUnmounted } from "vue";
import { ElMessage } from "element-plus";
import {
  getWikiList,
  createWiki,
  updateWiki,
  deleteWiki,
  likeWiki,
} from "@/api/modules/wiki";

const LS_MY_WIKI_IDS = "yuzuriha:wiki:my_ids";
const myWikiIds: string[] = JSON.parse(
  localStorage.getItem(LS_MY_WIKI_IDS) || "[]"
);
const markAsMine = (id: string | number) => {
  if (!myWikiIds.includes(String(id))) {
    myWikiIds.push(String(id));
    localStorage.setItem(LS_MY_WIKI_IDS, JSON.stringify(myWikiIds));
  }
};
const canEdit = (id: string | number) => myWikiIds.includes(String(id));

const entries = ref<any[]>([]);

const LS_LIKED_IDS = "yuzuriha:wiki:liked_ids";
const likedIds = ref<string[]>(
  JSON.parse(localStorage.getItem(LS_LIKED_IDS) || "[]")
);

const showModal = ref(false);
const editing = ref(false);
const editingId = ref<string | number | null>(null);
const detailEntry = ref<any>(null);
const form = reactive({ title: "", slug: "", author: "", content: "" });
const search = ref("");

function formatTime(ts: string | number | null | undefined) {
  if (!ts) return "未知时间";
  const date = new Date(ts);
  if (isNaN(date.getTime())) return "未知时间";
  return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(
    2,
    "0"
  )}-${String(date.getDate()).padStart(2, "0")}`;
}

async function loadEntries() {
  try {
    const res: any = await getWikiList();
    entries.value = res.data.map((e: any) => ({
      ...e,
      createdAt: e.createdAt || e.created_at,
      updatedAt: e.updatedAt || e.updated_at,
    }));
  } catch (err) {
    console.error(err);
    ElMessage.error("加载词条失败");
  }
}

function openCreate() {
  editing.value = false;
  editingId.value = null;
  form.title = "";
  form.slug = "";
  form.author = "";
  form.content = "";
  showModal.value = true;
}
function openEdit(entry: any) {
  if (!canEdit(entry.id)) {
    ElMessage.warning("只有创建者可以编辑");
    return;
  }
  editing.value = true;
  editingId.value = entry.id;
  form.title = entry.title;
  form.slug = entry.slug;
  form.author = entry.author;
  form.content = entry.content;
  showModal.value = true;
}
function closeModal() {
  showModal.value = false;
}

const canSubmit = computed(() => form.title.trim() && form.content.trim());

async function submit() {
  if (!canSubmit.value) {
    ElMessage.warning("请填写标题和内容");
    return;
  }
  const payload = {
    title: form.title.trim(),
    author: form.author.trim() || "匿名",
    content: form.content.trim(),
    slug: null,
  };
  if (form.slug.trim()) payload.slug = form.slug.trim();
  try {
    if (editing.value && editingId.value) {
      await updateWiki(editingId.value, payload);
      ElMessage.success("编辑成功");
    } else {
      const res: any = await createWiki(payload);
      markAsMine(res.data.id);
      editingId.value = res.data.id;
      ElMessage.success("创建成功");
    }
    showModal.value = false;
    loadEntries();
  } catch (err) {
    console.error(err);
    ElMessage.error("提交失败");
  }
}

async function remove(id: string | number) {
  if (!canEdit(id)) {
    ElMessage.warning("只有创建者可以删除");
    return;
  }
  if (!confirm("确认删除该词条？此操作不可撤销")) return;
  try {
    await deleteWiki(id);
    const index = myWikiIds.indexOf(String(id));
    if (index !== -1) myWikiIds.splice(index, 1);
    localStorage.setItem(LS_MY_WIKI_IDS, JSON.stringify(myWikiIds));
    ElMessage.success("删除成功");
    loadEntries();
  } catch (err) {
    console.error(err);
    ElMessage.error("删除失败");
  }
}

function persistLikedIds() {
  try {
    localStorage.setItem(LS_LIKED_IDS, JSON.stringify(likedIds.value));
  } catch (e) {
    console.warn("保存 likedIds 失败", e);
  }
}

function isLiked(id: string | number) {
  return likedIds.value.includes(String(id));
}

async function toggleLike(id: string | number) {
  const entry = entries.value.find((e) => e.id === id);
  if (!entry) return;

  const idStr = String(id);
  const wasLiked = likedIds.value.includes(idStr);

  if (wasLiked) {
    entry.likes = Math.max(0, (entry.likes || 0) - 1);
    likedIds.value = likedIds.value.filter((x) => x !== idStr);
  } else {
    entry.likes = (entry.likes || 0) + 1;
    likedIds.value.push(idStr);
  }
  persistLikedIds();

  try {
    const action = wasLiked ? "unlike" : "like";
    await likeWiki(id, action);
  } catch (err) {
    console.error("toggleLike error", err);
    if (wasLiked) {
      entry.likes = (entry.likes || 0) + 1;
      if (!likedIds.value.includes(idStr)) likedIds.value.push(idStr);
    } else {
      entry.likes = Math.max(0, (entry.likes || 0) - 1);
      likedIds.value = likedIds.value.filter((x) => x !== idStr);
    }
    persistLikedIds();
    ElMessage.error("点赞失败，请稍后重试");
  }
}

async function openDetail(entry: any) {
  detailEntry.value = entry;
}

const filteredEntries = computed(() => {
  const q = String(search.value || "")
    .trim()
    .toLowerCase();
  let list = entries.value;

  if (q) {
    list = list.filter(
      (e) =>
        (e.title || "").toLowerCase().includes(q) ||
        (e.slug || "").toLowerCase().includes(q)
    );
  }

  return [...list].sort((a, b) => (b.likes || 0) - (a.likes || 0));
});

const pcModules = import.meta.glob("@/assets/images1/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const mobileModules = import.meta.glob(
  "@/assets/images2/*.{jpg,png,jpeg,webp}",
  { eager: true }
);

const pcSrcs: string[] = Object.values(pcModules).map((m: any) => m.default);
const mobileSrcs: string[] = Object.values(mobileModules).map(
  (m: any) => m.default
);

function shuffle<T>(arr: T[]): T[] {
  const a = arr.slice();
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}

const randomFive = ref<string[]>([]);
const currentIndex = ref(0);
let timer: number;

function pickImages() {
  const isMobile = window.innerWidth < 768;
  const all = isMobile ? mobileSrcs : pcSrcs;
  randomFive.value = shuffle(all).slice(0, 5);
}

onMounted(() => {
  loadEntries();
  pickImages();
  window.addEventListener("resize", pickImages);

  timer = window.setInterval(() => {
    if (randomFive.value.length > 0) {
      currentIndex.value = (currentIndex.value + 1) % randomFive.value.length;
    }
  }, 5000);
});

onUnmounted(() => {
  clearInterval(timer);
  window.removeEventListener("resize", pickImages);
});
</script>

<style scoped lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;800&family=Rajdhani:wght@300;500;600&family=Share+Tech+Mono&display=swap");

/* 莫宁配色：深空机械 / 冰蓝光效 / 红瞳点缀 */
$bg-deep: #0a0c16;
$bg-panel: #121826;
$accent-blue: #4aa5ff;
$accent-light: #9ac9ff;
$red-pupil: #ff6b8b;
$text-primary: #f0f6ff;
$text-muted: rgba(240, 246, 255, 0.6);

.morning-wiki {
  position: relative;
  min-height: 100vh;
  color: $text-primary;
  padding: 16px;
  padding-top: 80px;
  background: radial-gradient(ellipse at 30% 40%, #0e1424, $bg-deep);
  font-family: "Rajdhani", "Inter", "PingFang SC", sans-serif;
  overflow: hidden;

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
      background: rgba(10, 12, 22, 0.65);
      backdrop-filter: blur(2px);
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
    z-index: 1;
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
    z-index: 1;
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
    height: 1px;
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

  /* 头部 */
  .wiki-header {
    position: relative;
    z-index: 10;
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 12px;
    padding: 16px 20px;
    background: rgba(10, 15, 30, 0.5);
    backdrop-filter: blur(12px);
    border: 1px solid rgba(74, 165, 255, 0.2);
    border-radius: 40px;
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.6);
    margin-bottom: 20px;
    flex-wrap: wrap;

    .title {
      h1 {
        margin: 0;
        font-family: "Orbitron", sans-serif;
        font-size: 1.5rem;
        background: linear-gradient(135deg, #fff, $accent-light);
        background-clip: text;
        -webkit-background-clip: text;
        color: transparent;
        text-shadow: 0 0 15px $accent-blue;
      }
      .subtitle {
        margin: 0;
        font-size: 0.9rem;
        color: $text-muted;
      }
    }

    .actions {
      display: flex;
      gap: 12px;
      align-items: center;
      flex-wrap: wrap;
    }

    .search {
      padding: 10px 16px;
      border-radius: 40px;
      border: 1px solid rgba(74, 165, 255, 0.3);
      background: rgba(0, 0, 0, 0.3);
      color: $text-primary;
      font-size: 0.9rem;
      outline: none;
      transition: border-color 0.2s, box-shadow 0.2s;

      &:focus {
        border-color: $accent-blue;
        box-shadow: 0 0 20px $accent-blue;
      }
      &::placeholder {
        color: $text-muted;
      }
    }

    .btn-new {
      background: linear-gradient(135deg, $accent-blue, $accent-light);
      color: $bg-deep;
      border: none;
      border-radius: 40px;
      padding: 10px 24px;
      font-size: 0.95rem;
      font-weight: 600;
      cursor: pointer;
      box-shadow: 0 10px 20px rgba($accent-blue, 0.3);
      transition: transform 0.2s, box-shadow 0.2s;

      &:hover {
        transform: translateY(-3px);
        box-shadow: 0 20px 30px rgba($accent-blue, 0.5);
      }
    }
  }

  /* 主体 */
  .wiki-body {
    position: relative;
    z-index: 10;

    .empty {
      text-align: center;
      padding: 40px;
      color: $text-muted;
    }

    .entry-list {
      list-style: none;
      padding: 0;
      margin: 0;
      display: grid;
      gap: 16px;

      .entry-card {
        background: rgba(10, 15, 30, 0.5);
        backdrop-filter: blur(8px);
        border: 1px solid rgba(74, 165, 255, 0.2);
        border-radius: 24px;
        padding: 16px;
        box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
        transition: transform 0.2s, box-shadow 0.2s, border-color 0.2s;

        &:hover {
          transform: translateY(-4px);
          box-shadow: 0 20px 40px rgba(0, 0, 0, 0.7);
          border-color: rgba(74, 165, 255, 0.5);
        }

        .entry-head {
          display: flex;
          justify-content: space-between;
          align-items: flex-start;
          gap: 12px;
          flex-wrap: wrap;

          .entry-meta {
            flex: 1;
            cursor: pointer;

            .entry-title-wrap {
              display: flex;
              align-items: center;
              gap: 8px;
              margin-bottom: 8px;

              .entry-title {
                margin: 0;
                font-size: 1.1rem;
                color: $accent-light;
                font-weight: 600;
              }
              .entry-badge {
                background: rgba(74, 165, 255, 0.15);
                border: 1px solid rgba(74, 165, 255, 0.3);
                border-radius: 30px;
                padding: 2px 10px;
                font-size: 0.8rem;
                color: $accent-light;
              }
            }

            .entry-info {
              display: flex;
              gap: 12px;
              flex-wrap: wrap;

              .meta-item {
                font-size: 0.85rem;
                color: $text-muted;
                background: rgba(0, 0, 0, 0.2);
                padding: 2px 10px;
                border-radius: 30px;
              }
            }
          }

          .entry-actions {
            display: flex;
            gap: 12px;
            align-items: center;

            .like {
              display: flex;
              align-items: center;
              gap: 4px;
              background: transparent;
              border: none;
              cursor: pointer;
              padding: 6px 12px;
              border-radius: 30px;
              transition: background 0.2s, transform 0.2s;

              img {
                width: 20px;
                height: 20px;
                filter: drop-shadow(0 0 4px $red-pupil);
              }

              .like-count {
                color: $text-primary;
                font-size: 0.9rem;
              }

              &.active {
                transform: scale(1.05);
                .like-count {
                  color: $red-pupil;
                  font-weight: 600;
                }
              }

              &:hover {
                background: rgba(74, 165, 255, 0.1);
              }
            }

            .edit-delete {
              display: flex;
              gap: 6px;

              .small {
                background: transparent;
                border: 1px solid rgba(74, 165, 255, 0.3);
                border-radius: 30px;
                padding: 6px 16px;
                color: $text-primary;
                font-size: 0.85rem;
                cursor: pointer;
                transition: all 0.2s;

                &:hover {
                  background: rgba(74, 165, 255, 0.1);
                  border-color: $accent-blue;
                }

                &.danger {
                  border-color: rgba(255, 107, 139, 0.3);
                  color: $red-pupil;

                  &:hover {
                    background: rgba(255, 107, 139, 0.1);
                  }
                }
              }
            }
          }
        }
      }
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
    z-index: 100;

    .modal {
      width: min(600px, 94%);
      max-height: 90vh;
      overflow-y: auto;
      background: $bg-panel;
      border: 1px solid rgba(74, 165, 255, 0.3);
      border-radius: 30px;
      padding: 24px;
      box-shadow: 0 30px 60px rgba(0, 0, 0, 0.8);

      .modal-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 20px;

        h3 {
          margin: 0;
          font-family: "Orbitron", sans-serif;
          color: $accent-light;
        }

        .close {
          background: transparent;
          border: none;
          color: $text-muted;
          font-size: 1.2rem;
          cursor: pointer;

          &:hover {
            color: $accent-light;
          }
        }
      }

      .modal-body {
        display: flex;
        flex-direction: column;
        gap: 16px;

        label {
          display: flex;
          flex-direction: column;
          gap: 6px;
          color: $text-primary;

          input,
          textarea {
            background: rgba(0, 0, 0, 0.3);
            border: 1px solid rgba(74, 165, 255, 0.3);
            border-radius: 30px;
            padding: 10px 16px;
            color: $text-primary;
            font-size: 0.95rem;
            outline: none;

            &:focus {
              border-color: $accent-blue;
              box-shadow: 0 0 20px $accent-blue;
            }
          }

          textarea {
            border-radius: 20px;
            resize: vertical;
          }
        }

        .detail-content {
          white-space: pre-wrap;
          line-height: 1.6;
        }
      }

      .modal-footer {
        display: flex;
        justify-content: flex-end;
        gap: 12px;
        margin-top: 24px;

        .btn {
          padding: 10px 24px;
          border: none;
          border-radius: 40px;
          font-weight: 600;
          cursor: pointer;
          transition: transform 0.2s, box-shadow 0.2s;

          &:not(.ghost) {
            background: linear-gradient(135deg, $accent-blue, $accent-light);
            color: $bg-deep;
            box-shadow: 0 10px 20px rgba($accent-blue, 0.3);

            &:hover {
              transform: translateY(-3px);
              box-shadow: 0 20px 30px rgba($accent-blue, 0.5);
            }
          }

          &.ghost {
            background: transparent;
            border: 1px solid rgba(74, 165, 255, 0.3);
            color: $text-primary;

            &:hover {
              background: rgba(74, 165, 255, 0.1);
            }
          }
        }
      }
    }
  }

  /* 过渡动画 */
  .fade-zoom-enter-active,
  .fade-zoom-leave-active {
    transition: opacity 0.3s, transform 0.3s;
  }
  .fade-zoom-enter-from,
  .fade-zoom-leave-to {
    opacity: 0;
    transform: scale(0.95);
  }

  /* 移动端响应式 */
  @media (max-width: 768px) {
    .wiki-header {
      flex-direction: column;
      align-items: stretch;
    }

    .entry-actions {
    
      width: 100%;

      .edit-delete {
        width: 100%;
        button {
          flex: 1;
        }
      }
    }

    .modal {
      padding: 16px;
    }
  }
}
</style>
