<template>
  <div class="morning-resources">
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

    <!-- 静态装饰：六边形网格背景（极淡） -->
    <div class="hex-bg"></div>
    <div class="grid-bg"></div>

    <!-- 头部 -->
    <header class="hero">
      <div class="hero-inner">
        <h1>演构资源</h1>
        <p class="subtitle">可自由上传关于莫宁的相关链接</p>
      </div>
    </header>

    <main class="container">
      <!-- 上传区（可折叠） -->
      <section class="uploader" :class="{ collapsed: uploaderCollapsed }">
        <div class="uploader-head">
          <button
            class="toggle"
            @click="toggleUploader"
            :aria-expanded="!uploaderCollapsed"
          >
            <span v-if="uploaderCollapsed">展开上传区</span>
            <span v-else>收起上传区</span>
          </button>
        </div>

        <form
          @submit.prevent="addResource"
          class="upload-form"
          :aria-hidden="uploaderCollapsed"
        >
          <div class="row">
            <input
              v-model="form.title"
              type="text"
              placeholder="标题（必填，如有解压码请注明）"
              aria-label="标题"
            />
            <input
              v-model="form.type"
              type="text"
              placeholder="链接类型（网页/视频/网盘等）"
              aria-label="类型"
            />
          </div>

          <div class="row">
            <input
              v-model="form.uploader"
              type="text"
              placeholder="上传人（可选）"
              aria-label="上传人"
            />
            <input
              v-model="form.link"
              type="url"
              placeholder="链接（以 https:// 开头）"
              aria-label="链接"
            />
          </div>

          <div class="actions">
            <button type="submit" class="btn primary">上传</button>
          </div>
        </form>
      </section>

      <!-- 资源列表 -->
      <section class="list">
        <div class="list-header">
          <h2>资源列表（{{ resources.length }}）</h2>
          <div class="sort">
            <label>
              排序：
              <select v-model="sortBy">
                <option value="time">按时间（新→旧）</option>
                <option value="likes">按点赞（高→低）</option>
              </select>
            </label>
          </div>
        </div>

        <ul class="items">
          <li v-for="item in sortedResources" :key="item.id" class="item">
            <a
              :href="item.link"
              target="_blank"
              rel="noopener noreferrer"
              class="title"
              >{{ item.title }}</a
            >

            <div class="meta">
              <div class="left">
                <span class="uploader">{{ item.uploader || "匿名" }}</span>
                <span class="dot">•</span>
                <time :datetime="item.time">{{ formatTime(item.time) }}</time>
              </div>

              <div class="right">
                <button
                  @click.prevent="handleLike(item)"
                  :aria-pressed="likedIds.has(String(item.id))"
                  class="like-btn"
                  :class="{ active: likedIds.has(String(item.id)) }"
                >
                  <img
                    :src="
                      likedIds.has(String(item.id))
                        ? '/icons/heart-red-filled.svg'
                        : '/icons/heart-red-outline.svg'
                    "
                    class="heart-icon"
                    alt="heart"
                  />
                  <span class="count">{{ item.likes }}</span>
                </button>

                <span class="badge">{{ item.type }}</span>
              </div>
            </div>
          </li>
        </ul>

        <p v-if="resources.length === 0" class="empty">
          目前没有资源，快来上传第一条吧！
        </p>
      </section>
    </main>

    <footer class="foot">提示：点击标题将直接跳转</footer>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount } from "vue";
import {
  getResourceList,
  createResource,
  likeResource,
} from "@/api/modules/resource";
import { ElMessage } from "element-plus";

interface Resource {
  id: number | string;
  title: string;
  uploader?: string;
  time: string;
  likes: number;
  link: string;
  type: string;
  role_key?: string;
}

const STORAGE_KEY = "fll_resources_v1";
const DEFAULT_ROLE = "moning";

const form = ref<{
  title: string;
  uploader: string;
  link: string;
  type: string;
}>({
  title: "",
  uploader: "",
  link: "",
  type: "",
});

const resources = ref<Resource[]>([]);
const likedIds = ref(new Set<string>());
const sortBy = ref<"time" | "likes">("time");
const uploaderCollapsed = ref(false);

function mapServerToLocal(row: any): Resource {
  return {
    id: row.id,
    title: row.title,
    uploader: row.uploader || "匿名",
    time: row.created_at || row.time || new Date().toISOString(),
    likes: row.likes ?? 0,
    link: row.link,
    type: row.storage_type || row.type || "其他",
    role_key: row.role_key,
  };
}

async function loadResources() {
  try {
    const res: any = await getResourceList({
      role_key: DEFAULT_ROLE,
      page: 1,
      pageSize: 100,
    });
    if (res && res.success && Array.isArray(res.data)) {
      resources.value = res.data.map(mapServerToLocal);
      const raw = localStorage.getItem(STORAGE_KEY);
      if (raw) {
        try {
          const parsed = JSON.parse(raw);
          if (parsed.liked && Array.isArray(parsed.liked)) {
            parsed.liked.forEach((id: string) => likedIds.value.add(id));
          }
        } catch (e) {}
      }
      return;
    }
  } catch (err) {
    console.warn("拉取资源失败，使用本地缓存", err);
  }
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (raw) {
      const parsed = JSON.parse(raw);
      if (parsed.liked && Array.isArray(parsed.liked)) {
        parsed.liked.forEach((id: string) => likedIds.value.add(id));
      }
    }
  } catch (e) {}
}

function saveLocalCache() {
  try {
    const liked = Array.from(likedIds.value);
    localStorage.setItem(STORAGE_KEY, JSON.stringify({ liked }));
  } catch (e) {}
}

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
let Imgtimer: number | undefined;

onMounted(() => {
  loadResources();
  Imgtimer = window.setInterval(() => {
    currentIndex.value =
      (currentIndex.value + 1) % Math.max(1, randomFive.value.length);
  }, 5200);
  uploaderCollapsed.value = window.innerWidth <= 640;
});
function toggleUploader() {
  uploaderCollapsed.value = !uploaderCollapsed.value;
}
onBeforeUnmount(() => {
  if (Imgtimer) clearInterval(Imgtimer);
});

async function addResource() {
  const t = form.value.title.trim();
  const l = form.value.link.trim();
  if (!form.value.title.trim() || !form.value.link.trim()) {
    return ElMessage.warning("请填写完整信息");
  }
  if (!/^https?:\/\//i.test(l)) {
    return ElMessage.error("请输入正确的链接(https开头)");
  }
  try {
    const payload = {
      title: t,
      uploader: form.value.uploader.trim() || "匿名",
      link: l,
      storage_type: form.value.type,
      role_key: DEFAULT_ROLE,
    };
    const res: any = await createResource(payload);
    if (res && res.success && res.data) {
      const added = mapServerToLocal(res.data);
      resources.value.unshift(added);
      saveLocalCache();
      resetForm();
      ElMessage.success("上传成功");
      return;
    }
    ElMessage.error("上传失败");
  } catch (err) {
    console.warn("创建资源失败", err);
  }
}

function resetForm() {
  form.value.title = "";
  form.value.uploader = "";
  form.value.link = "";
  form.value.type = "";
}

async function handleLike(item: Resource) {
  const id = item.id;
  const wasLiked = likedIds.value.has(String(id));
  if (wasLiked) {
    likedIds.value.delete(String(id));
    item.likes = Math.max(0, item.likes - 1);
  } else {
    likedIds.value.add(String(id));
    item.likes++;
  }
  saveLocalCache();

  try {
    const action = wasLiked ? "unlike" : "like";
    const res: any = await likeResource(id, action);
    if (
      res &&
      res.success &&
      res.data &&
      typeof res.data.likes !== "undefined"
    ) {
      item.likes = res.data.likes;
    }
  } catch (err) {
    console.warn("点赞接口调用失败，回滚本地状态", err);
    if (wasLiked) {
      likedIds.value.add(String(id));
      item.likes++;
    } else {
      likedIds.value.delete(String(id));
      item.likes = Math.max(0, item.likes - 1);
    }
    saveLocalCache();
  }
}

const sortedResources = computed(() => {
  const arr = [...resources.value];
  if (sortBy.value === "time") {
    arr.sort((a, b) => +new Date(b.time) - +new Date(a.time));
  } else {
    arr.sort((a, b) => b.likes - a.likes);
  }
  return arr;
});

function formatTime(iso: string) {
  try {
    const d = new Date(iso);
    return new Intl.DateTimeFormat("zh-CN", {
      month: "2-digit",
      day: "2-digit",
      hour: "2-digit",
      minute: "2-digit",
    }).format(d);
  } catch (e) {
    return iso;
  }
}
</script>

<style scoped lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;800&family=Rajdhani:wght@300;500;600&family=Share+Tech+Mono&display=swap");

/* 莫宁配色：深空机械 / 冰蓝光效 / 红瞳点缀（仅心形） */
$bg-deep: #0a0c16; // 深空底色
$bg-panel: #121826; // 面板深色
$accent-blue: #4aa5ff; // 冰蓝主色
$accent-light: #9ac9ff; // 浅蓝高光
$red-pupil: #ff6b8b; // 红瞳（心形用）
$text-primary: #f0f6ff; // 冷白文字
$text-muted: rgba(240, 246, 255, 0.6);

.morning-resources {
  color: $text-primary;
  display: flex;
  flex-direction: column;
  padding-top: 60px;
  font-family: "Rajdhani", "Inter", "PingFang SC", sans-serif;
  background: radial-gradient(ellipse at 30% 40%, #0e1424, $bg-deep);
  min-height: 100vh;
  position: relative;
  overflow: hidden;

  /* 背景轮播（冷色滤镜） */
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

  /* 头部 */
  .hero {
    position: relative;
    z-index: 10;
    padding: 18px 12px;
    background: rgba(10, 15, 30, 0.5);
    backdrop-filter: blur(2px);
    border-bottom: 1px solid rgba(74, 165, 255, 0.2);
    margin-bottom: 20px;

    .hero-inner {
      max-width: 1000px;
      margin: 0 auto;

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

      .subtitle {
        margin-top: 4px;
        font-size: 1rem;
        color: $text-muted;
      }
    }
  }

  /* 主容器 */
  .container {
    position: relative;
    z-index: 10;
    max-width: 1000px;
    margin: 0 auto;
    padding: 0 12px;
    width: 100%;
    box-sizing: border-box;

    /* 上传区卡片 */
    .uploader {
      border-radius: 24px;
      margin-bottom: 20px;
      background: rgba(10, 15, 30, 0.5);
      backdrop-filter: blur(2px);
      border: 1px solid rgba(74, 165, 255, 0.2);
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
      transition: all 0.3s;

      .uploader-head {
        display: flex;
        justify-content: flex-end;
        padding: 12px 16px;

        .toggle {
          background: transparent;
          border: 1px solid rgba(74, 165, 255, 0.3);
          border-radius: 30px;
          padding: 6px 16px;
          color: $accent-light;
          font-size: 0.9rem;
          cursor: pointer;
          transition: background 0.2s, box-shadow 0.2s;

          &:hover {
            background: rgba(74, 165, 255, 0.1);
            box-shadow: 0 0 15px $accent-blue;
          }
        }
      }

      .upload-form {
        padding: 16px;
        max-height: 500px;
        overflow: hidden;
        transition: max-height 0.3s ease, padding 0.3s ease;

        .row {
          display: flex;
          gap: 12px;
          margin-bottom: 12px;

          input {
            flex: 1;
            padding: 12px 16px;
            background: rgba(0, 0, 0, 0.3);
            border: 1px solid rgba(74, 165, 255, 0.3);
            border-radius: 40px;
            color: $text-primary;
            font-size: 0.95rem;
            outline: none;
            transition: border-color 0.2s, box-shadow 0.2s;

            &:focus {
              border-color: $accent-blue;
              box-shadow: 0 0 20px $accent-blue;
            }
            &::placeholder {
              color: rgba(240, 246, 255, 0.3);
            }
          }
        }

        .actions {
          display: flex;
          justify-content: flex-end;

          .btn {
            padding: 10px 30px;
            border: none;
            border-radius: 40px;
            font-weight: 600;
            cursor: pointer;
            transition: transform 0.2s, box-shadow 0.2s;

            &.primary {
              background: linear-gradient(135deg, $accent-blue, $accent-light);
              color: $bg-deep;
              box-shadow: 0 10px 20px rgba($accent-blue, 0.3);

              &:hover {
                transform: translateY(-3px);
                box-shadow: 0 20px 30px rgba($accent-blue, 0.5);
              }
            }
          }
        }
      }

      &.collapsed {
        .upload-form {
          max-height: 0;
          padding-top: 0;
          padding-bottom: 0;
        }
      }
    }

    /* 资源列表 */
    .list {
      .list-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 16px;

        h2 {
          margin: 0;
          font-family: "Orbitron", sans-serif;
          font-size: 1.3rem;
          color: $accent-light;
          text-shadow: 0 0 10px $accent-blue;
        }

        .sort {
          select {
            padding: 8px 16px;
            background: rgba(0, 0, 0, 0.3);
            border: 1px solid rgba(74, 165, 255, 0.3);
            border-radius: 40px;
            color: $text-primary;
            font-size: 0.9rem;
            outline: none;
            cursor: pointer;
            transition: border-color 0.2s;

            &:focus {
              border-color: $accent-blue;
            }
          }
        }
      }

      .items {
        list-style: none;
        padding: 0;
        margin: 0;
        max-height: 60vh;
        overflow-y: auto;

        &::-webkit-scrollbar {
          width: 6px;
        }
        &::-webkit-scrollbar-thumb {
          background: $accent-blue;
          border-radius: 10px;
        }
      }

      .item {
        background: rgba(10, 15, 30, 0.5);
        backdrop-filter: blur(2px);
        border: 1px solid rgba(74, 165, 255, 0.2);
        border-radius: 16px;
        padding: 16px;
        margin-bottom: 12px;
        transition: transform 0.2s, box-shadow 0.2s, border-color 0.2s;

        &:hover {
          transform: translateY(-4px);
          box-shadow: 0 20px 40px rgba(0, 0, 0, 0.6);
          border-color: rgba(74, 165, 255, 0.5);
        }

        .title {
          display: block;
          color: $text-primary;
          font-size: 1.1rem;
          font-weight: 600;
          text-decoration: none;
          margin-bottom: 12px;
          word-break: break-word;
          transition: color 0.2s;

          &:hover {
            color: $accent-light;
            text-decoration: underline;
          }
        }

        .meta {
          display: flex;
          justify-content: space-between;
          align-items: center;
          font-size: 0.9rem;

          .left {
            display: flex;
            align-items: center;
            gap: 8px;

            .uploader {
              color: $accent-light;
              font-weight: 600;
              margin: auto 0;
              padding: 6px;
            }

            .dot {
              color: $text-muted;
            }

            time {
              color: $text-muted;
            }
          }

          .right {
            display: flex;
            align-items: center;
            gap: 12px;

            .like-btn {
              display: flex;
              align-items: center;
              gap: 4px;
              background: transparent;
              border: none;
              cursor: pointer;
              padding: 4px 8px;
              border-radius: 30px;
              transition: background 0.2s, transform 0.2s;

              &:hover {
                background: rgba(74, 165, 255, 0.1);
                transform: scale(1.05);
              }

              .heart-icon {
                width: 18px;
                height: 18px;
                filter: drop-shadow(0 0 4px $red-pupil);
              }

              .count {
                color: $text-primary;
                font-size: 0.9rem;
                min-width: 20px;
                text-align: center;
              }

              &.active .count {
                color: $red-pupil;
                font-weight: 600;
              }
            }

            .badge {
              padding: 4px 12px;
              background: rgba(74, 165, 255, 0.15);
              border: 1px solid rgba(74, 165, 255, 0.3);
              border-radius: 30px;
              color: $accent-light;
              font-size: 0.8rem;
              font-weight: 500;
            }
          }
        }
      }

      .empty {
        text-align: center;
        color: $text-muted;
        padding: 40px 0;
      }
    }
  }

  /* 底部提示 */
  .foot {
    position: relative;
    z-index: 10;
    text-align: center;
    color: $text-muted;
    font-size: 0.85rem;
    margin: 40px 0 20px;
  }

  /* 移动端响应式 */
  @media (max-width: 640px) {
    .carousel1 {
      display: none;
    }
    .carousel2 {
      display: block;
    }

    .hero .hero-inner h1 {
      font-size: 1.4rem;
    }

    .upload-form .row {
      flex-direction: column;
      gap: 8px;
    }

    .list-header {
      flex-direction: column;
      align-items: flex-start;
      gap: 8px;
    }

    .item .meta {
      flex-direction: column;
      align-items: flex-start;
      gap: 8px;
    }
  }
}
</style>
