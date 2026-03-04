<template>
  <div class="gallery-container">
    <button class="upload-btn" @click="openUploadModal">上传图片</button>

    <section class="gallery section">
      <div class="sort-controls">
        <button @click="toggleSort" class="sort-btn">
          按 {{ sortBy === "like_count" ? "点赞量" : "最新上传" }} 排序
        </button>
      </div>
      <div class="gallery-grid">
        <div
          v-for="(img, index) in images"
          :key="img.id"
          class="card"
          @click="openLightbox(index)"
          ref="cards"
        >
          <div class="card-inner">
            <img
              :src="img.src"
              :alt="img.alt"
              loading="lazy"
              @load="onImageLoad($event)"
            />
            <div class="overlay">
              <span>查看大图</span>
            </div>
            <button class="like-btn" @click.stop="handleLike(img)">
              <i class="heart" :class="{ liked: img.liked }"></i>
              <span class="like-count">{{ img.likeCount }}</span>
            </button>
          </div>
        </div>
      </div>
      <!-- sentinel：用于触发无限滚动 -->
      <div ref="sentinel" class="sentinel"></div>
      <!-- 可选：加载中/结束提示 -->
      <div class="loading" v-if="loading">加载中...</div>
      <div class="finished" v-if="finished">已全部加载</div>
    </section>

    <aside class="ranking-panel">
      <div class="panel-header" @click="expanded = !expanded">
        <h3 class="ranking-title">演算排行</h3>
        <span>共{{ imgTotal }}张</span>
        <span class="toggle-icon">{{ expanded ? "▾" : "▸" }}</span>
      </div>
      <transition name="fade">
        <ul v-if="expanded" class="ranking-list">
          <li
            v-for="(item, idx) in rankingList"
            :key="idx"
            class="ranking-item"
            :class="`rank-${idx + 1}`"
          >
            <span class="rank">{{ idx + 1 }}</span>
            <span class="name">{{ item.nickname }}</span>
            <span class="count">{{ item.count }} 张</span>
          </li>
        </ul>
      </transition>
    </aside>

    <!-- Lightbox Modal -->
    <div v-if="lightboxOpen" class="lightbox" @click.self="closeLightbox">
      <span class="close" @click="closeLightbox">✕</span>
      <span class="prev" @click.stop="prevImage">‹</span>
      <img :src="images[currentIndex].src" :alt="images[currentIndex].alt" />
      <span class="next" @click.stop="nextImage">›</span>
    </div>

    <!-- 上传弹窗 -->
    <div
      v-if="uploadModalOpen"
      class="upload-modal-overlay"
      @click.self="closeUploadModal"
    >
      <div class="upload-modal">
        <h3>批量上传图片</h3>
        <div class="tip-container">
          <ul class="tips-list">
            <li>
              审核规则： 1.不要色情倾向（不要露三点，我怕被封）
              2.要我能认出是莫宁。
            </li>
            <li>
              由于没有用户系统，我这边不好做审核反馈，但只要显示上传成功，我这边肯定能收到。
            </li>
            <li>
              如果图片数量较多请在b站私信联系我给我网盘链接，因为我云服务器比较小一次性上传太多图片可能会导致上传不上，感谢理解。
            </li>
            <li>
              因为审核上传一次比较麻烦，所以审核时间不定，最晚一周，感谢谅解。
            </li>
          </ul>
        </div>
        <p class="stats">
          今日已上传：<strong>{{ uploadedToday }}</strong> 张，
          剩余可上传：<strong>{{ remaining }}</strong> 张
        </p>
        <label>
          昵称：
          <input v-model="nickname" type="text" placeholder="请输入昵称" />
        </label>
        <label>
          选择图片（最多 {{ remaining }} 张）：
          <input
            ref="fileInput"
            type="file"
            multiple
            accept="image/*"
            @change="handleFileSelect"
          />
        </label>
        <p class="tip" v-if="selectedFiles.length">
          已选 {{ selectedFiles.length }} 张
        </p>
        <div class="modal-actions">
          <button :disabled="!canSubmit || isUploading" @click="submitUpload">
            {{ isUploading ? "上传中..." : "立即上传" }}
          </button>
          <button class="cancel" @click="closeUploadModal">取消</button>
        </div>
      </div>
    </div>

    <div class="floating-chibis">
      <img
        v-for="(pet, i) in chibiList"
        :key="i"
        :src="pet.src"
        :style="{ top: pet.top + 'px', left: pet.left + 'px' }"
        class="chibi-img"
      />
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, computed, nextTick, onBeforeUnmount } from "vue";
import { uploadImages } from "@/api/modules/images";
import { getRankingList } from "@/api/modules/ranking";
import { gsap } from "gsap";
import { getImagesLikesList, likeImage } from "@/api/modules/imagesLikes";
import { debounce } from "lodash";

const sortBy = ref<"uploaded_at" | "like_count">("like_count");
const order = ref<"asc" | "desc">("desc");
function toggleSort() {
  if (sortBy.value === "uploaded_at") {
    sortBy.value = "like_count";
    order.value = "desc";
  } else {
    sortBy.value = "uploaded_at";
    order.value = "desc";
  }
  pageImage.value = 1;
  images.value = [];
  finished.value = false;
  window.scrollTo(0, 0);
  loadNextPage();
}
// 获取已点赞 ID 数组
function getLikedIds(): number[] {
  const data = localStorage.getItem("likedImageIds");
  return data ? JSON.parse(data) : [];
}

// 保存已点赞 ID 数组
function setLikedIds(ids: number[]) {
  localStorage.setItem("likedImageIds", JSON.stringify(ids));
}

async function handleLike(img: ImageItem) {
  if (img.liked) return;

  try {
    await likeImage(img.id);
    img.likeCount += 1;
    img.liked = true;

    const likedIds = getLikedIds();
    likedIds.push(img.id);
    setLikedIds(likedIds);
  } catch (error) {
    console.error("点赞失败", error);
    alert("点赞失败，请稍后重试");
  }
}

interface ImageItem {
  src: string;
  alt: string;
  likeCount: number;
  id: number;
  liked: Boolean;
}

interface RankingItem {
  id?: number;
  nickname: string;
  count: number;
}
const rankingList = ref<RankingItem[]>([]);
const expanded = ref(true);

const page = 1;
const pageSize = 99;

const fetchRanking = async () => {
  const res = await getRankingList({
    page,
    pageSize,
    character_key: "moning",
  });
  if (res.success) {
    rankingList.value = res.data;
  } else {
    console.error("获取排行榜失败", res.message);
  }
};

const images = ref<ImageItem[]>([]);

const pageImage = ref(1);
const limit = ref(10);
const loading = ref(false);
const finished = ref(false);

const sentinel = ref<HTMLElement | null>(null);

const observerCard = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        entry.target.classList.add("visible");
        observerCard.unobserve(entry.target);
      }
    });
  },
  { threshold: 0.1 }
);

async function observeNewCards(startIndex = 0) {
  await nextTick();
  const cards = document.querySelectorAll<HTMLElement>(".card");
  for (let i = startIndex; i < cards.length; i++) {
    observerCard.observe(cards[i]);
  }
}
const imgTotal = ref(0);
async function loadNextPage() {
  if (loading.value || finished.value) return;
  loading.value = true;
  try {
    const res = await getImagesLikesList({
      page: pageImage.value,
      limit: limit.value,
      sortBy: sortBy.value,
      character_key: "moning",
      order: order.value,
    });
    imgTotal.value = res.total;
    const likedIds = getLikedIds();
    const list = (
      res.images as Array<{ url: string; like_count: number; id: number }>
    ).map((item) => ({
      src: item.url,
      alt: "",
      likeCount: item.like_count,
      id: item.id,
      liked: likedIds.includes(item.id),
    }));
    if (list.length === 0) {
      finished.value = true;
      return;
    }
    const oldLength = images.value.length;
    const existingIds = new Set(images.value.map((i) => i.id));
    const filtered = list.filter((item) => !existingIds.has(item.id));
    images.value.push(...filtered);
    pageImage.value++;

    observeNewCards(oldLength);
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
}

const debouncedLoad = debounce(
  () => {
    loadNextPage();
  },
  200,
  { leading: true, trailing: false }
);

const lightboxOpen = ref(false);
const currentIndex = ref(0);

function openLightbox(index: number) {
  currentIndex.value = index;
  lightboxOpen.value = true;
}
function closeLightbox() {
  lightboxOpen.value = false;
}
function prevImage() {
  currentIndex.value =
    (currentIndex.value + images.value.length - 1) % images.value.length;
}
function nextImage() {
  currentIndex.value = (currentIndex.value + 1) % images.value.length;
}

function onImageLoad(e: Event) {
  const img = e.target as HTMLImageElement;
  const card = img.closest(".card");
  card?.classList.add("loaded");
}

const uploadModalOpen = ref(false);
const nickname = ref("");
const fileInput = ref<HTMLInputElement>();
const selectedFiles = ref<File[]>([]);

function getTodayKey() {
  return `uploaded_${new Date().toISOString().slice(0, 10)}`;
}
const uploadedToday = ref<number>(
  Number(localStorage.getItem(getTodayKey()) || 0)
);
const remaining = computed(() => Math.max(27 - uploadedToday.value, 0));

const canSubmit = computed(() => {
  return (
    nickname.value.trim().length > 0 &&
    selectedFiles.value.length > 0 &&
    selectedFiles.value.length <= remaining.value
  );
});

function clearOldUploadRecords() {
  const today = new Date();
  const storage = window.localStorage;
  for (const key of Object.keys(storage)) {
    if (!key.startsWith("uploaded_")) continue;

    const dateStr = key.slice("uploaded_".length);
    const recordDate = new Date(dateStr);
    if (isNaN(recordDate.getTime())) continue;

    const diffMs = today.getTime() - recordDate.getTime();
    const diffDays = diffMs / (1000 * 60 * 60 * 24);

    if (diffDays > 2) {
      storage.removeItem(key);
    }
  }
}

function openUploadModal() {
  clearOldUploadRecords();
  nickname.value = "";
  selectedFiles.value = [];
  if (fileInput.value) fileInput.value.value = "";
  uploadedToday.value = Number(localStorage.getItem(getTodayKey()) || 0);
  uploadModalOpen.value = true;
}
function closeUploadModal() {
  uploadModalOpen.value = false;
}

function handleFileSelect(e: Event) {
  const files = Array.from((e.target as HTMLInputElement).files || []);

  if (!files) return;

  const validFiles: File[] = [];
  for (const file of files) {
    if (file.size > 20 * 1024 * 1024) {
      alert(`文件太大：${file.name}，请控制在 20MB 内`);
      continue;
    }
    validFiles.push(file);
  }

  if (validFiles.length === 0) return;

  if (validFiles.length > remaining.value) {
    alert(
      `今天最多还能上传 ${remaining.value} 张，已为你截取前 ${remaining.value} 张`
    );
    selectedFiles.value = files.slice(0, remaining.value);
  } else {
    selectedFiles.value = files;
  }
}
const isUploading = ref(false);
async function submitUpload() {
  if (!canSubmit.value) return;
  isUploading.value = true;
  try {
    const res = await uploadImages(
      selectedFiles.value,
      nickname.value.trim(),
      "moning"
    );
    const uploadedCount = res.data.length;
    uploadedToday.value += uploadedCount;
    localStorage.setItem(getTodayKey(), String(uploadedToday.value));

    alert(`成功上传 ${uploadedCount} 张图片`);
    closeUploadModal();
  } catch (err: any) {
    console.error(err);
    alert(err.message || "上传失败");
  } finally {
    isUploading.value = false;
  }
}

interface Chibi {
  src: string;
  top: number;
  left: number;
}

const chibiList = ref<Chibi[]>([]);
let sentinelObserver: IntersectionObserver;

onMounted(async () => {
  await fetchRanking();

  await loadNextPage();
  observeNewCards(0);

  sentinelObserver = new IntersectionObserver(
    (entries) => {
      if (entries[0].isIntersecting) debouncedLoad();
    },
    { rootMargin: "0px", threshold: 0.1 }
  );
  if (sentinel.value) {
    sentinelObserver.observe(sentinel.value);
  }
  const total = 6;
  let pickCount = 3;
  const vw = window.innerWidth;
  const vh = window.innerHeight;
  const isMobile = window.innerWidth <= 768;
  const imgWidth = 100;
  const imgHeight = 100;

  function shuffle(array) {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
    return array;
  }

  if (isMobile) {
    pickCount = 1;
  }
  const nums = shuffle(Array.from({ length: total }, (_, k) => k + 1));
  const picks = nums.slice(0, pickCount);

  chibiList.value = [];
  picks.forEach((i) => {
    chibiList.value.push({
      src: `/QImages/1 (${i}).png`,
      left: Math.random() * (vw - imgWidth),
      top: Math.random() * (vh - imgHeight),
    });
  });

  await nextTick();

  const imgs = document.querySelectorAll<HTMLImageElement>(".chibi-img");
  imgs.forEach((img, index) => {
    const padding = 200;
    gsap.fromTo(
      img,
      { opacity: 0, scale: 0.5 },
      {
        opacity: 1,
        scale: 1,
        duration: 0.8,
        ease: "back.out(2)",
        delay: 0.2 * index,
      }
    );

    img.addEventListener("mouseenter", () => {
      gsap.killTweensOf(img);

      gsap.to(img, {
        x: "+=" + ((Math.random() - 0.5) * 400).toFixed(0),
        y: "+=" + ((Math.random() - 0.5) * 400).toFixed(0),
        duration: 1.2,
        ease: "back.out(2)",
        onComplete: () => {
          animate(img);
        },
      });
    });

    const animate = (img: HTMLImageElement) => {
      let { x, y } = img.getBoundingClientRect();
      let deltaX = (Math.random() - 0.5) * 200;
      let deltaY = (Math.random() - 0.5) * 200;

      let nextX = x + deltaX;
      let nextY = y + deltaY;

      if (nextX < padding) deltaX = padding - x;
      if (nextX + img.width > window.innerWidth - padding)
        deltaX = window.innerWidth - padding - (x + img.width);
      if (nextY < padding) deltaY = padding - y;
      if (nextY + img.height > window.innerHeight - padding)
        deltaY = window.innerHeight - padding - (y + img.height);

      gsap.to(img, {
        x: `+=${deltaX.toFixed(0)}`,
        y: `+=${deltaY.toFixed(0)}`,
        rotation: `+=${((Math.random() - 0.5) * 60).toFixed(0)}`,
        duration: 2 + Math.random() * 2,
        ease: "power1.inOut",
        onComplete: () => animate(img),
      });
    };
    animate(img);
  });
});

onBeforeUnmount(() => {
  observerCard.disconnect();
  sentinelObserver.disconnect();
});
</script>

<style scoped lang="scss">
/* 莫宁风格配色：深空机械 / 冰蓝光效 */
$bg-deep: #0a0c16; // 深空底色
$bg-panel: #121826; // 面板深色
$accent-blue: #4aa5ff; // 冰蓝主色
$accent-light: #9ac9ff; // 浅蓝高光
$red-pupil: #ff6b8b; // 红瞳点缀（仅用于心形）
$text-primary: #f0f6ff; // 冷白文字
$card-bg: rgba(10, 15, 30, 0.5);
$card-border: rgba(74, 165, 255, 0.2);
$soft-shadow: 0 10px 30px rgba(0, 0, 0, 0.7);

@import url("https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;800&family=Rajdhani:wght@300;500;600&family=Share+Tech+Mono&display=swap");

.gallery-container {
  background: radial-gradient(circle at 30% 40%, #0e1424, $bg-deep);
  color: $text-primary;
  min-height: 100vh;
  padding-bottom: 60px;
  padding-top: 20px;
  font-family: "Rajdhani", "Inter", "PingFang SC", sans-serif;

  .section {
    padding: 80px 20px;
    max-width: 1200px;
    margin: 0 auto;

    .sort-controls {
      margin: 16px 0;

      .sort-btn {
        display: inline-flex;
        align-items: center;
        gap: 8px;
        padding: 10px 28px 10px 56px;
        font-size: 1rem;
        font-family: "Rajdhani", sans-serif;
        cursor: pointer;
        border-radius: 28px;
        position: relative;
        overflow: hidden;
        border: 1px solid rgba($accent-light, 0.2);
        background: rgba(10, 15, 30, 0.7);
        backdrop-filter: blur(8px);
        color: $text-primary;
        box-shadow: 0 10px 28px rgba(0, 0, 0, 0.6),
          inset 0 1px 0 rgba($accent-blue, 0.1);
        transition: transform 0.2s, box-shadow 0.2s, background 0.2s;

        &::after {
          content: "";
          position: absolute;
          left: 18px;
          top: 50%;
          transform: translateY(-50%);
          width: 20px;
          height: 20px;
          background: linear-gradient(135deg, $accent-blue, $accent-light);
          clip-path: polygon(50% 0, 100% 25%, 100% 75%, 50% 100%, 0 75%, 0 25%);
          box-shadow: 0 0 12px $accent-blue;
        }

        &:hover {
          transform: translateY(-4px);
          box-shadow: 0 20px 40px rgba(0, 0, 0, 0.8),
            0 0 30px rgba($accent-blue, 0.3);
          background: rgba(15, 20, 35, 0.8);
        }

        &:active {
          transform: translateY(-2px);
        }
      }
    }

    .gallery-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
      gap: 24px;

      .card {
        perspective: 1000px;
        opacity: 0;
        transform: translateY(20px);

        &.visible {
          animation: fadeInUp 0.6s ease forwards;
        }

        &.loaded {
          .card-inner img {
            filter: none;
            opacity: 1;
          }
        }

        .card-inner {
          position: relative;
          border-radius: 16px;
          overflow: hidden;
          box-shadow: $soft-shadow;
          transform-style: preserve-3d;
          transition: transform 0.4s ease, box-shadow 0.4s ease;
          border: 1px solid $card-border;

          &:hover {
            transform: rotateY(4deg) rotateX(2deg) scale(1.03);
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.9),
              0 0 30px rgba($accent-blue, 0.3);
          }

          img {
            width: 100%;
            height: 100%;
            object-fit: cover;
            display: block;
            filter: blur(20px) grayscale(40%);
            opacity: 0.8;
            transition: filter 0.6s ease, opacity 0.6s ease;
          }

          .overlay {
            position: absolute;
            bottom: 0;
            width: 100%;
            padding: 12px 0;
            background: linear-gradient(transparent, rgba(0, 0, 0, 0.8));
            text-align: center;
            opacity: 0;
            transition: opacity 0.4s;

            span {
              color: $text-primary;
              font-family: "Orbitron", sans-serif;
              font-size: 1rem;
              letter-spacing: 1px;
              background: rgba(0, 0, 0, 0.5);
              padding: 4px 16px;
              border-radius: 30px;
              border: 1px solid rgba($accent-blue, 0.3);
            }
          }

          &:hover .overlay {
            opacity: 1;
          }

          .like-btn {
            position: absolute;
            bottom: 12px;
            right: 12px;
            background: transparent;
            border: none;
            cursor: pointer;
            z-index: 2;
            display: flex;
            align-items: center;
            gap: 6px;
            padding: 4px;
            border-radius: 50%;
            transition: transform 0.2s ease;

            &:hover {
              transform: scale(1.2);
            }

            .heart {
              width: 24px;
              height: 24px;
              background: url("/icons/heart-red-outline.svg") no-repeat center;
              background-size: contain;
              transition: all 0.3s ease;
              filter: drop-shadow(0 0 6px $accent-blue);
            }

            .liked {
              background: url("/icons/heart-red-filled.svg") no-repeat center;
              background-size: contain;
              animation: pop 0.4s ease;

              &::after {
                content: "";
                position: absolute;
                top: 50%;
                left: 50%;
                width: 40px;
                height: 40px;
                background: rgba($accent-blue, 0.3);
                border-radius: 50%;
                transform: translate(-50%, -50%) scale(0);
                animation: pulse 1.2s ease-out infinite;
                pointer-events: none;
              }
            }

            .like-count {
              font-size: 1rem;
              color: $red-pupil;
              text-shadow: 0 0 8px $accent-blue;
              font-weight: bold;
            }
          }

          @keyframes pulse {
            0% {
              transform: translate(-50%, -50%) scale(0.6);
              opacity: 0.6;
            }
            50% {
              transform: translate(-50%, -50%) scale(1.2);
              opacity: 0;
            }
            100% {
              transform: translate(-50%, -50%) scale(0.6);
              opacity: 0;
            }
          }
        }
      }
    }
  }

  .lightbox {
    position: fixed;
    inset: 0;
    background: rgba(2, 6, 10, 0.96);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;

    img {
      max-width: 85%;
      max-height: 85%;
      border-radius: 8px;
      box-shadow: 0 8px 24px rgba(0, 0, 0, 0.9);
      animation: fadeInUp 0.4s ease;
    }

    .close,
    .prev,
    .next {
      position: absolute;
      color: $text-primary;
      font-size: 2.5rem;
      cursor: pointer;
      user-select: none;
      padding: 8px;
      border-radius: 50%;
      transition: color 0.2s, background 0.2s;

      &:hover {
        color: $accent-light;
        background: rgba(255, 255, 255, 0.1);
      }
    }

    .close {
      top: 20px;
      right: 20px;
    }
    .prev {
      left: 20px;
      top: 50%;
      transform: translateY(-50%);
    }
    .next {
      right: 20px;
      top: 50%;
      transform: translateY(-50%);
    }
  }

  .upload-btn {
    position: fixed;
    bottom: 64px;
    left: 24px;
    display: inline-flex;
    align-items: center;
    gap: 10px;
    padding: 12px 24px;
    font-size: 1rem;
    z-index: 10;
    cursor: pointer;
    color: $bg-deep;
    background: linear-gradient(135deg, $accent-blue, $accent-light);
    border: none;
    border-radius: 40px;
    box-shadow: 0 10px 30px rgba($accent-blue, 0.4);
    transition: transform 0.2s, box-shadow 0.2s;
    font-weight: 600;

    &:hover {
      transform: translateY(-4px);
      box-shadow: 0 20px 40px rgba($accent-blue, 0.6);
    }

    &:active {
      transform: translateY(-2px);
    }
  }

  .upload-modal-overlay {
    position: fixed;
    inset: 0;
    z-index: 2000;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(0, 0, 0, 0.8);
    backdrop-filter: blur(8px);
  }

  .upload-modal {
    width: 720px;
    max-width: calc(100% - 40px);
    padding: 36px;
    border-radius: 24px;
    background: $bg-panel;
    border: 1px solid rgba($accent-blue, 0.3);
    box-shadow: 0 30px 60px rgba(0, 0, 0, 0.8),
      inset 0 1px 0 rgba($accent-light, 0.1);
    color: $text-primary;

    h3 {
      margin: 0 0 20px;
      font-size: 1.8rem;
      font-family: "Orbitron", sans-serif;
      text-align: center;
      background: linear-gradient(135deg, #fff, $accent-light);
      background-clip: text;
      -webkit-background-clip: text;
      color: transparent;
    }

    .tip-container {
      background: rgba(0, 0, 0, 0.3);
      border-left: 4px solid $accent-blue;
      padding: 16px;
      border-radius: 12px;
      margin-bottom: 20px;

      .tips-list {
        list-style: none;
        padding: 0;
        margin: 0;

        li {
          position: relative;
          padding-left: 24px;
          margin-bottom: 8px;
          font-size: 0.95rem;
          color: rgba($text-primary, 0.9);

          &::before {
            content: "";
            position: absolute;
            left: 4px;
            top: 6px;
            width: 8px;
            height: 8px;
            background: $accent-blue;
            border-radius: 50%;
            box-shadow: 0 0 10px $accent-blue;
          }
        }
      }
    }

    .stats {
      font-size: 1rem;
      text-align: center;
      margin: 20px 0;
      strong {
        color: $accent-light;
      }
    }

    label {
      display: block;
      margin-bottom: 20px;
      font-size: 1rem;

      input {
        width: 100%;
        margin-top: 8px;
        padding: 12px 16px;
        background: rgba(0, 0, 0, 0.4);
        border: 1px solid rgba($accent-blue, 0.3);
        border-radius: 40px;
        color: $text-primary;
        font-size: 1rem;
        outline: none;
        transition: border-color 0.2s, box-shadow 0.2s;

        &:focus {
          border-color: $accent-blue;
          box-shadow: 0 0 20px $accent-blue;
        }

        &::placeholder {
          color: rgba($text-primary, 0.3);
        }
      }
    }

    .modal-actions {
      display: flex;
      justify-content: flex-end;
      gap: 16px;
      margin-top: 30px;

      button {
        padding: 12px 28px;
        border: none;
        border-radius: 40px;
        font-weight: 600;
        cursor: pointer;
        transition: transform 0.2s, box-shadow 0.2s;

        &:not(.cancel) {
          background: linear-gradient(135deg, $accent-blue, $accent-light);
          color: $bg-deep;
          box-shadow: 0 10px 20px rgba($accent-blue, 0.3);

          &:hover:not(:disabled) {
            transform: translateY(-3px);
            box-shadow: 0 20px 30px rgba($accent-blue, 0.5);
          }

          &:disabled {
            opacity: 0.5;
            cursor: not-allowed;
          }
        }

        &.cancel {
          background: transparent;
          border: 1px solid rgba($accent-light, 0.3);
          color: $text-primary;

          &:hover {
            background: rgba(255, 255, 255, 0.05);
          }
        }
      }
    }
  }

  .ranking-panel {
    width: 240px;
    padding: 16px;
    position: fixed;
    top: 80px;
    right: 20px;
    z-index: 100;
    background: rgba(10, 15, 30, 0.8);
    backdrop-filter: blur(12px);
    border-radius: 24px;
    border: 1px solid rgba($accent-blue, 0.2);
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.6);

    .panel-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      cursor: pointer;
      padding-bottom: 8px;

      .ranking-title {
        margin: 0;
        font-size: 1.2rem;
        font-family: "Orbitron", sans-serif;
        color: $accent-light;
        letter-spacing: 1px;
      }

      .toggle-icon {
        font-size: 1.2rem;
        color: $accent-blue;
        background: rgba(0, 0, 0, 0.3);
        padding: 4px 10px;
        border-radius: 20px;
        transition: background 0.2s;

        &:hover {
          background: rgba($accent-blue, 0.2);
        }
      }
    }

    .ranking-list {
      list-style: none;
      padding: 0;
      margin: 12px 0 0;
      max-height: 50vh;
      overflow-y: auto;

      &::-webkit-scrollbar {
        width: 6px;
      }
      &::-webkit-scrollbar-thumb {
        background: $accent-blue;
        border-radius: 10px;
      }
    }

    .ranking-item {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 10px 12px;
      margin-bottom: 8px;
      border-radius: 12px;
      background: rgba(0, 0, 0, 0.3);
      transition: transform 0.2s, background 0.2s;

      &:hover {
        transform: translateX(-4px);
        background: rgba($accent-blue, 0.15);
      }

      .rank {
        width: 32px;
        text-align: center;
        font-weight: 800;
        color: $accent-light;
      }
      .name {
        flex: 1;
        padding: 0 8px;
        font-size: 0.95rem;
        color: $text-primary;
        font-weight: 500;
      }
      .count {
        font-size: 0.9rem;
        color: $accent-blue;
        font-weight: 600;
      }

      &.rank-1 {
        background: linear-gradient(
          90deg,
          rgba($accent-blue, 0.3),
          rgba($accent-light, 0.1)
        );
        border-left: 4px solid $accent-light;
        .rank,
        .name,
        .count {
          color: #fff;
          text-shadow: 0 0 10px $accent-blue;
        }
      }
      &.rank-2 {
        border-left: 4px solid $accent-blue;
      }
      &.rank-3 {
        border-left: 4px solid rgba($accent-blue, 0.5);
      }
    }
  }

  .floating-chibis {
    position: fixed;
    inset: 0;
    pointer-events: none;
    z-index: 1;

    .chibi-img {
      position: absolute;
      width: 80px;
      user-select: none;
      pointer-events: auto;
      z-index: 10;
      filter: drop-shadow(0 0 10px $accent-blue);
    }
  }

  .sentinel {
    height: 20px;
    width: 100%;
  }

  .loading,
  .finished {
    text-align: center;
    padding: 30px;
    font-family: "Share Tech Mono", monospace;
    color: $accent-light;
    text-shadow: 0 0 10px currentColor;
  }
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes pop {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.3);
  }
  100% {
    transform: scale(1);
  }
}

/* 响应式 */
@media (max-width: 768px) {
  .ranking-panel {
    width: 200px;
    top: 60px;
    right: 10px;
  }
  .upload-btn {
    bottom: 30px;
    left: 16px;
    padding: 8px 16px;
  }
}
</style>
