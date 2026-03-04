<template>
  <div class="morning-match">
    <!-- 装饰性元素 -->
    <div class="glow-orb glow-1"></div>
    <div class="glow-orb glow-2"></div>
    <div class="grid-bg"></div>
    <div class="hex-bg"></div>
    <div class="scan-line"></div>
    <div class="data-flow"></div>
    <div class="floating-particles">
      <span v-for="n in 8" :key="n" class="particle"></span>
    </div>

    <!-- 游戏头部 -->
    <div class="game-header">
      <div class="title-area">
        <img :src="avatarSrc" alt="莫宁头像" class="avatar" />
        <h2>星星消消乐 · 星枢演构</h2>
      </div>
      <div class="score-area">
        静质量能
        <transition name="pop" mode="out-in">
          <span :key="score" class="score-value">{{ score }}</span>
        </transition>
      </div>
    </div>

    <!-- 昵称输入和排行榜按钮 -->
    <div class="user-panel">
      <div class="nickname-wrapper">
        <span class="label">昵称</span>
        <input
          v-model="nickname"
          type="text"
          placeholder="输入昵称"
          class="nickname-input"
          maxlength="20"
        />
      </div>
      <button class="rank-btn" @click="openRankingDrawer">
        <span>🏆</span> 排行榜
      </button>
    </div>

    <!-- 游戏网格 -->
    <div class="grid-container">
      <div class="grid-row" v-for="(row, rowIndex) in grid" :key="rowIndex">
        <div
          v-for="(cell, colIndex) in row"
          :key="`${rowIndex}-${colIndex}`"
          class="grid-cell"
          :class="{
            selected: selectedRow === rowIndex && selectedCol === colIndex,
            'can-swap': canSwap(rowIndex, colIndex),
            'swap-flash': isSwapping(rowIndex, colIndex),
            'swap-back': isSwappingBack(rowIndex, colIndex),
            eliminate: isEliminating(rowIndex, colIndex),
            appear: isAppearing(rowIndex, colIndex),
          }"
          @click="onCellClick(rowIndex, colIndex)"
          @animationend="onAnimationEnd(rowIndex, colIndex, $event)"
        >
          <img :src="avatars[cell]" class="star-icon" />
        </div>
      </div>
    </div>

    <!-- 死局提示 -->
    <transition name="fade">
      <div v-if="showDeadlockTip" class="deadlock-tip">
        ⚠️ 无解 · 重新演算中
      </div>
    </transition>

    <!-- 角色技能栏 -->
    <div class="character-skill-bar">
      <div
        v-for="(char, index) in characters"
        :key="char.name"
        class="skill-item"
        :class="{ 'skill-ready': char.count >= 30 }"
        @click="activateSkill(index)"
      >
        <div class="skill-icon-wrapper">
          <img :src="char.avatar" :alt="char.name" class="skill-avatar" />
        
          <div class="skill-count">{{ char.count }}/30</div>
        </div>
        <div class="skill-name">{{ char.name }}</div>
        <div class="skill-effect">{{ char.effect }}</div>
        <div v-if="char.cooldown > 0" class="skill-cooldown">
          {{ char.cooldown }}s
        </div>
      </div>
    </div>

    <!-- 技能特效文字 -->
    <transition name="fade-scale">
      <div v-if="skillEffectText" class="skill-effect-text">
        {{ skillEffectText }}
      </div>
    </transition>

    <!-- 操作提示 -->
    <div class="game-tip">
      点击选中一颗星星，再点击相邻星星交换。消除越多得分越高！消除30个对应角色可释放技能。
    </div>

    <!-- 提交分数按钮 -->
    <div class="submit-area">
      <button
        class="submit-btn"
        :disabled="submitting || !canSubmit"
        @click="submitScore"
      >
        {{ submitting ? "提交中..." : "提交静质量能" }}
      </button>
    </div>

    <!-- 排行榜抽屉 -->
    <transition name="slide">
      <div v-if="rankingDrawerOpen" class="ranking-drawer">
        <div class="drawer-header">
          <h3>⭐ 演算排行榜 ⭐</h3>
          <button class="close-btn" @click="rankingDrawerOpen = false">
            ✕
          </button>
        </div>
        <div class="ranking-list" v-if="rankingList.length">
          <div
            v-for="(item, idx) in rankingList"
            :key="item.id || idx"
            class="ranking-item"
            :class="{ 'self-rank': item.nickname === nickname }"
          >
            <span class="rank">{{ idx + 1 }}</span>
            <span class="name">{{ item.nickname }}</span>
            <span class="score">{{ item.count }}</span>
          </div>
        </div>
        <div v-else class="empty-rank">暂无数据，快来上榜吧~</div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, computed, watch, onBeforeUnmount } from "vue";
import {
  getRankingList,
  addRankingItem,
  updateRankingItem,
} from "@/api/modules/ranking";

// 头像路径
const avatarSrc = new URL("@/assets/avatar/avatar1.png", import.meta.url).href;

// 头像图片数组 (5种)
const avatars = [
  new URL("@/assets/avatar/avatar1.png", import.meta.url).href, // 莫宁
  new URL("@/assets/avatar/avatar2.png", import.meta.url).href, // 漂泊者♀
  new URL("@/assets/avatar/avatar3.png", import.meta.url).href, // 漂泊者♂
  new URL("@/assets/avatar/avatar4.png", import.meta.url).href, // 琳奈
  new URL("@/assets/avatar/avatar5.png", import.meta.url).href, // 爱弥斯
];

// 游戏配置
const ROWS = 6;
const COLS = 6;

// 游戏状态
const grid = ref([]);
const score = ref(0);
const selectedRow = ref(null);
const selectedCol = ref(null);
const nickname = ref(localStorage.getItem("morning_match_nickname") || "");
const submitting = ref(false);
const rankingDrawerOpen = ref(false);
const rankingList = ref([]);
const CHARACTER_KEY = "morning_match";
const showDeadlockTip = ref(false);

// 存储所有技能冷却定时器，用于组件卸载时清理
const skillTimers = [];

// 技能特效文字
const skillEffectText = ref("");
const showSkillEffect = (text) => {
  skillEffectText.value = text;
  setTimeout(() => (skillEffectText.value = ""), 2000);
};

// 动画状态集合
const swapSet = ref(new Set());
const swapBackSet = ref(new Set());
const eliminateSet = ref(new Set());
const appearSet = ref(new Set());

function isSwapping(row, col) {
  return swapSet.value.has(`${row},${col}`);
}
function isSwappingBack(row, col) {
  return swapBackSet.value.has(`${row},${col}`);
}
function isEliminating(row, col) {
  return eliminateSet.value.has(`${row},${col}`);
}
function isAppearing(row, col) {
  return appearSet.value.has(`${row},${col}`);
}

function onAnimationEnd(row, col, event) {
  const key = `${row},${col}`;
  if (event.animationName === "swapPulse") {
    swapSet.value.delete(key);
  } else if (event.animationName === "swapBackAnim") {
    swapBackSet.value.delete(key);
  } else if (event.animationName === "eliminateAnim") {
    eliminateSet.value.delete(key);
  } else if (event.animationName === "appearAnim") {
    appearSet.value.delete(key);
  }
}

// 角色技能配置
const characters = ref([
  {
    name: "莫宁",
    avatar: avatars[0],
    count: 0,
    cooldown: 0,
    effect: "直接加10分",
    skill: function () {
      score.value += 10;
      // 全屏闪烁动画
      document.querySelectorAll(".grid-cell").forEach((cell) => {
        cell.style.animation = "skillHeal 1.2s ease";
        setTimeout(() => (cell.style.animation = ""), 1200);
      });
      showSkillEffect("立场展开 · +10分");
    },
  },
  {
    name: "守岸人",
    avatar: avatars[1],
    count: 0,
    cooldown: 0,
    effect: "随机消除6个",
    skill: function () {
      const nonEmpty = [];
      for (let r = 0; r < ROWS; r++) {
        for (let c = 0; c < COLS; c++) {
          if (grid.value[r][c] !== null) nonEmpty.push([r, c]);
        }
      }
      const shuffled = nonEmpty.sort(() => Math.random() - 0.5);
      const selected = shuffled.slice(0, Math.min(6, nonEmpty.length));
      processElimination(selected);
      showSkillEffect("宣告终约 · 随机消除6个");
    },
  },
  {
    name: "漂泊者",
    avatar: avatars[2],
    count: 0,
    cooldown: 0,
    effect: "随机消除一列",
    skill: function () {
      const col = Math.floor(Math.random() * COLS);
      const positions = [];
      for (let row = 0; row < ROWS; row++) {
        if (grid.value[row][col] !== null) positions.push([row, col]);
      }
      processElimination(positions);
      showSkillEffect("湮灭万律 · 整列消除");
    },
  },
  {
    name: "琳奈",
    avatar: avatars[3],
    count: 0,
    cooldown: 0,
    effect: "同类消除",
    skill: function () {
      const targetType = Math.floor(Math.random() * avatars.length);
      const positions = [];
      for (let r = 0; r < ROWS; r++) {
        for (let c = 0; c < COLS; c++) {
          if (grid.value[r][c] === targetType) positions.push([r, c]);
        }
      }
      processElimination(positions);
      showSkillEffect("喷涂！ · 同类消除");
    },
  },
  {
    name: "爱弥斯",
    avatar: avatars[4],
    count: 0,
    cooldown: 0,
    effect: "随机消除一行",
    skill: function () {
      const row = Math.floor(Math.random() * ROWS);
      const positions = [];
      for (let c = 0; c < COLS; c++) {
        if (grid.value[row][c] !== null) positions.push([row, c]);
      }
      processElimination(positions);
      showSkillEffect("贯穿吧！ · 整行消除");
    },
  },
]);

// 统一处理消除（技能调用）
function processElimination(positions) {
  // 如果没有要消除的格子，直接返回，避免触发重力
  if (positions.length === 0) return;

  positions.forEach(([r, c]) => {
    eliminateSet.value.add(`${r},${c}`);
    const charIndex = grid.value[r][c];
    if (charIndex !== null) {
      characters.value[charIndex].count = Math.min(
        characters.value[charIndex].count + 1,
        30
      );
      score.value += 1;
    }
  });

  setTimeout(() => {
    positions.forEach(([r, c]) => {
      eliminateSet.value.delete(`${r},${c}`);
      grid.value[r][c] = null;
    });
    applyGravityWithAnimation(true);
  }, 800);
}

// 带动画的重力
function applyGravityWithAnimation(isFromSkill = false) {
  for (let j = 0; j < COLS; j++) {
    const column = [];
    for (let i = ROWS - 1; i >= 0; i--) {
      if (grid.value[i][j] !== null) {
        column.push(grid.value[i][j]);
      }
    }
    for (let i = ROWS - 1; i >= 0; i--) {
      if (column.length > 0) {
        grid.value[i][j] = column.shift();
      } else {
        const newVal = Math.floor(Math.random() * avatars.length);
        grid.value[i][j] = newVal;
        appearSet.value.add(`${i},${j}`);
      }
    }
  }

  setTimeout(() => {
    const hasMoreMatches = processMatches(false, isFromSkill);
    if (!hasMoreMatches && !isFromSkill) {
      checkAndHandleDeadlock();
    }
  }, 600);
}

// 初始化网格
function initGrid() {
  const newGrid = [];
  for (let i = 0; i < ROWS; i++) {
    const row = [];
    for (let j = 0; j < COLS; j++) {
      row.push(Math.floor(Math.random() * avatars.length));
    }
    newGrid.push(row);
  }
  grid.value = newGrid;
  characters.value.forEach((char) => (char.count = 0));
  processMatches(true);
}
initGrid();

// 检测三消
function findMatches() {
  const matches = [];
  // 水平
  for (let i = 0; i < ROWS; i++) {
    let j = 0;
    while (j < COLS - 2) {
      const val = grid.value[i][j];
      if (val === null) {
        j++;
        continue;
      }
      let len = 1;
      while (j + len < COLS && grid.value[i][j + len] === val) len++;
      if (len >= 3) {
        for (let k = 0; k < len; k++) matches.push([i, j + k]);
      }
      j += len;
    }
  }
  // 垂直
  for (let j = 0; j < COLS; j++) {
    let i = 0;
    while (i < ROWS - 2) {
      const val = grid.value[i][j];
      if (val === null) {
        i++;
        continue;
      }
      let len = 1;
      while (i + len < ROWS && grid.value[i + len][j] === val) len++;
      if (len >= 3) {
        for (let k = 0; k < len; k++) matches.push([i + k, j]);
      }
      i += len;
    }
  }
  return matches;
}

// 消除并处理连锁
function processMatches(silent = false, isFromSkill = false) {
  const matches = findMatches();
  if (matches.length === 0) return false;

  const uniqueMatches = [...new Set(matches.map((p) => p.join(",")))].map((s) =>
    s.split(",").map(Number)
  );
  const points = uniqueMatches.length * 1;
  if (!silent) score.value += points;

  uniqueMatches.forEach(([r, c]) => {
    eliminateSet.value.add(`${r},${c}`);
    const charIndex = grid.value[r][c];
    if (charIndex !== null) {
      characters.value[charIndex].count = Math.min(
        characters.value[charIndex].count + 1,
        30
      );
    }
  });

  setTimeout(() => {
    uniqueMatches.forEach(([r, c]) => {
      eliminateSet.value.delete(`${r},${c}`);
      grid.value[r][c] = null;
    });
    applyGravityWithAnimation(isFromSkill);
  }, 600);

  return true;
}

// 交换动画
function swapWithAnimation(r1, c1, r2, c2) {
  swapSet.value.add(`${r1},${c1}`);
  swapSet.value.add(`${r2},${c2}`);

  setTimeout(() => {
    swapSet.value.delete(`${r1},${c1}`);
    swapSet.value.delete(`${r2},${c2}`);

    const temp = grid.value[r1][c1];
    grid.value[r1][c1] = grid.value[r2][c2];
    grid.value[r2][c2] = temp;

    if (findMatches().length > 0) {
      processMatches(false, false);
    } else {
      swapBackSet.value.add(`${r1},${c1}`);
      swapBackSet.value.add(`${r2},${c2}`);
      setTimeout(() => {
        const tempBack = grid.value[r1][c1];
        grid.value[r1][c1] = grid.value[r2][c2];
        grid.value[r2][c2] = tempBack;
        swapBackSet.value.delete(`${r1},${c1}`);
        swapBackSet.value.delete(`${r2},${c2}`);
      }, 400);
    }
  }, 400);
}

function isAdjacent(r1, c1, r2, c2) {
  return Math.abs(r1 - r2) + Math.abs(c1 - c2) === 1;
}

function canSwap(row, col) {
  if (selectedRow.value === null || selectedCol.value === null) return false;
  if (row === selectedRow.value && col === selectedCol.value) return false;
  return isAdjacent(selectedRow.value, selectedCol.value, row, col);
}

function onCellClick(row, col) {
  if (selectedRow.value === null || selectedCol.value === null) {
    selectedRow.value = row;
    selectedCol.value = col;
    return;
  }
  if (selectedRow.value === row && selectedCol.value === col) {
    selectedRow.value = null;
    selectedCol.value = null;
    return;
  }
  if (isAdjacent(selectedRow.value, selectedCol.value, row, col)) {
    const oldRow = selectedRow.value;
    const oldCol = selectedCol.value;
    selectedRow.value = null;
    selectedCol.value = null;
    swapWithAnimation(oldRow, oldCol, row, col);
  } else {
    selectedRow.value = row;
    selectedCol.value = col;
  }
}

function activateSkill(index) {
  const char = characters.value[index];
  if (char.count >= 30 && char.cooldown <= 0) {
    char.skill();
    char.count = 0;
    char.cooldown = 5;
    const timer = setInterval(() => {
      if (char.cooldown > 0) char.cooldown--;
      else clearInterval(timer);
    }, 1000);
    // 存储定时器以便组件卸载时清理
    skillTimers.push(timer);
  }
}

// 死局检测
function checkDeadlock() {
  for (let r1 = 0; r1 < ROWS; r1++) {
    for (let c1 = 0; c1 < COLS; c1++) {
      if (grid.value[r1][c1] === null) continue;
      // 右侧
      if (c1 + 1 < COLS && grid.value[r1][c1 + 1] !== null) {
        [grid.value[r1][c1], grid.value[r1][c1 + 1]] = [
          grid.value[r1][c1 + 1],
          grid.value[r1][c1],
        ];
        if (findMatches().length > 0) {
          [grid.value[r1][c1], grid.value[r1][c1 + 1]] = [
            grid.value[r1][c1 + 1],
            grid.value[r1][c1],
          ];
          return false;
        }
        [grid.value[r1][c1], grid.value[r1][c1 + 1]] = [
          grid.value[r1][c1 + 1],
          grid.value[r1][c1],
        ];
      }
      // 下方
      if (r1 + 1 < ROWS && grid.value[r1 + 1][c1] !== null) {
        [grid.value[r1][c1], grid.value[r1 + 1][c1]] = [
          grid.value[r1 + 1][c1],
          grid.value[r1][c1],
        ];
        if (findMatches().length > 0) {
          [grid.value[r1][c1], grid.value[r1 + 1][c1]] = [
            grid.value[r1 + 1][c1],
            grid.value[r1][c1],
          ];
          return false;
        }
        [grid.value[r1][c1], grid.value[r1 + 1][c1]] = [
          grid.value[r1 + 1][c1],
          grid.value[r1][c1],
        ];
      }
    }
  }
  return true;
}

function handleDeadlock() {
  showDeadlockTip.value = true;
  for (let i = 0; i < ROWS; i++) {
    for (let j = 0; j < COLS; j++) {
      grid.value[i][j] = Math.floor(Math.random() * avatars.length);
    }
  }
  processMatches(true);
  setTimeout(() => {
    showDeadlockTip.value = false;
  }, 2000);
}

function checkAndHandleDeadlock() {
  if (checkDeadlock()) handleDeadlock();
}

// 昵称存储
watch(nickname, (newVal) => {
  if (newVal) localStorage.setItem("morning_match_nickname", newVal);
});

const canSubmit = computed(() => nickname.value.trim() && score.value > 0);

// 排行榜
const page = 1;
const pageSize = 999;
async function fetchRanking() {
  try {
    const res = await getRankingList({
      page,
      pageSize,
      character_key: CHARACTER_KEY,
    });
    if (res?.success) rankingList.value = res.data || [];
    else rankingList.value = [];
  } catch {
    rankingList.value = [];
  }
}
function openRankingDrawer() {
  fetchRanking();
  rankingDrawerOpen.value = true;
}
async function submitScore() {
  if (!canSubmit.value) return;
  submitting.value = true;
  const payload = {
    character_key: CHARACTER_KEY,
    nickname: nickname.value.trim(),
    count: score.value,
  };
  if (!payload.nickname) return;
  const existing = rankingList.value.reduce(
    (acc, cur) =>
      cur.nickname === payload.nickname
        ? !acc || cur.count > acc.count
          ? cur
          : acc
        : acc,
    null
  );
  try {
    if (existing) {
      if (payload.count > existing.count) {
        await updateRankingItem(existing.id, { count: payload.count });
        await fetchRanking();
        rankingDrawerOpen.value = true;
      } else {
        rankingDrawerOpen.value = true;
      }
    } else {
      await addRankingItem(payload);
      await fetchRanking();
      rankingDrawerOpen.value = true;
    }
  } catch (err) {
    console.error(err);
  } finally {
    submitting.value = false;
  }
}

// 组件卸载前清除所有技能冷却定时器
onBeforeUnmount(() => {
  skillTimers.forEach((timer) => clearInterval(timer));
  skillTimers.length = 0;
});
</script>

<style scoped lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Orbitron:wght@400;600;800&family=Rajdhani:wght@300;500;600&display=swap");

$bg-deep: #0a0c16;
$bg-panel: #121826;
$accent-blue: #4aa5ff;
$accent-light: #9ac9ff;
$red-pupil: #ff6b8b;
$text-primary: #f0f6ff;

.morning-match {
  position: relative;
  background: radial-gradient(ellipse at 30% 40%, #0e1424, $bg-deep);
  min-height: 100vh;
  color: $text-primary;
  font-family: "Rajdhani", "Inter", sans-serif;
  padding: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  overflow-x: hidden;
  padding-top: 80px;
}

/* 装饰层（保持不变） */
.glow-orb {
  position: absolute;
  border-radius: 50%;
  filter: blur(60px);
  z-index: 0;
  opacity: 0.2;
  animation: glowPulse 10s infinite alternate;
}
.glow-1 {
  top: 10%;
  left: 5%;
  width: 400px;
  height: 400px;
  background: radial-gradient(circle, $accent-blue, transparent 70%);
}
.glow-2 {
  bottom: 10%;
  right: 5%;
  width: 500px;
  height: 500px;
  background: radial-gradient(circle, $accent-light, transparent 70%);
  animation-delay: 3s;
}
@keyframes glowPulse {
  0% {
    opacity: 0.1;
    transform: scale(1);
  }
  100% {
    opacity: 0.3;
    transform: scale(1.3);
  }
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
  animation: scanMove 8s linear infinite;
  pointer-events: none;
  z-index: 1;
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
  animation: flowMove 6s linear infinite;
  z-index: 1;
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
  opacity: 0.4;
  animation: floatParticle 20s infinite;
}
.particle:nth-child(1) {
  top: 10%;
  left: 20%;
  animation-delay: 0s;
}
.particle:nth-child(2) {
  top: 30%;
  left: 70%;
  animation-delay: 2s;
}
.particle:nth-child(3) {
  top: 50%;
  left: 40%;
  animation-delay: 4s;
}
.particle:nth-child(4) {
  top: 70%;
  left: 80%;
  animation-delay: 6s;
}
.particle:nth-child(5) {
  top: 20%;
  left: 90%;
  animation-delay: 8s;
}
.particle:nth-child(6) {
  top: 80%;
  left: 10%;
  animation-delay: 10s;
}
.particle:nth-child(7) {
  top: 40%;
  left: 30%;
  animation-delay: 12s;
}
.particle:nth-child(8) {
  top: 60%;
  left: 60%;
  animation-delay: 14s;
}
@keyframes floatParticle {
  0% {
    transform: translate(0, 0);
    opacity: 0.2;
  }
  25% {
    transform: translate(30px, -30px);
    opacity: 0.6;
  }
  50% {
    transform: translate(-20px, 40px);
    opacity: 0.3;
  }
  75% {
    transform: translate(20px, 20px);
    opacity: 0.5;
  }
  100% {
    transform: translate(0, 0);
    opacity: 0.2;
  }
}

/* 游戏头部 */
.game-header {
  position: relative;
  z-index: 10;
  width: 100%;
  max-width: 600px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;

  .title-area {
    display: flex;
    align-items: center;
    gap: 10px;
    .avatar {
      width: 40px;
      height: 40px;
      border-radius: 50%;
      border: 2px solid $accent-blue;
      box-shadow: 0 0 20px $accent-blue;
    }
    h2 {
      font-size: 1.4rem;
      font-weight: 600;
      letter-spacing: 1px;
      text-shadow: 0 0 10px $accent-blue;
    }
  }
  .score-area {
    font-size: 1.2rem;
    background: rgba(0, 0, 0, 0.3);
    padding: 8px 20px;
    border-radius: 40px;
    border: 1px solid rgba($accent-blue, 0.4);
    .score-value {
      color: $accent-light;
      font-size: 1.8rem;
      margin-left: 8px;
      font-weight: 600;
      display: inline-block;
    }
  }
}

/* 分数弹出动画 */
.pop-enter-active {
  animation: pop 0.2s ease;
}
.pop-leave-active {
  display: none;
}
@keyframes pop {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.5);
    color: white;
  }
  100% {
    transform: scale(1);
  }
}

/* 用户面板 */
.user-panel {
  position: relative;
  z-index: 10;
  width: 100%;
  max-width: 600px;
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  align-items: center;

  .nickname-wrapper {
    flex: 1;
    display: flex;
    align-items: center;
    background: rgba(0, 0, 0, 0.3);
    border: 1px solid rgba($accent-blue, 0.3);
    border-radius: 40px;
    padding: 4px 4px 4px 16px;

    .label {
      color: $accent-light;
      margin-right: 8px;
    }

    .nickname-input {
      flex: 1;
      background: transparent;
      border: none;
      color: white;
      font-size: 1rem;
      outline: none;
      &::placeholder {
        color: rgba(255, 255, 255, 0.3);
      }
    }
  }

  .rank-btn {
    background: rgba(10, 15, 30, 0.8);
    border: 1px solid $accent-blue;
    color: $accent-light;
    padding: 8px 20px;
    border-radius: 40px;
    animation: cursorAnimation_link 1s infinite step-start;
    display: flex;
    align-items: center;
    gap: 6px;
    transition: 0.2s;
    &:hover {
      background: rgba($accent-blue, 0.2);
      box-shadow: 0 0 20px $accent-blue;
    }
  }
}

/* 网格容器 */
.grid-container {
  position: relative;
  z-index: 10;
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin: 20px 0;
  background: rgba(0, 0, 0, 0.2);
  padding: 16px;
  border-radius: 32px;
  border: 1px solid rgba($accent-blue, 0.2);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.6);
}

.grid-row {
  display: flex;
  gap: 8px;
  justify-content: center;
}
.grid-cell {
  width: 70px;
  height: 70px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  animation: cursorAnimation_link 1s infinite step-start;
  transition: all 0.2s;
  box-shadow: 0 0 20px $accent-blue;
  border: 2px solid transparent;
  background: rgba(10, 15, 30, 0.6);
  overflow: hidden;
  &.selected {
    border-color: white;
    box-shadow: 0 0 30px white, 0 0 20px $accent-blue;
    transform: scale(1.05);
  }
  &.can-swap {
    filter: brightness(1.2);
    box-shadow: 0 0 30px $accent-light;
    animation: canSwapPulse 0.8s infinite alternate;
  }
  &.swap-flash {
    animation: swapPulse 0.4s ease;
  }
  &.swap-back {
    animation: swapBackAnim 0.4s ease;
  }
  &.eliminate {
    animation: eliminateAnim 0.6s ease forwards;
  }
  &.appear {
    animation: appearAnim 0.6s ease;
  }
  .star-icon {
    width: 100%;
    height: 100%;
    object-fit: cover;
    border-radius: 50%;
    border: 2px solid $accent-blue;
  }
}

/* 动画定义 */
@keyframes canSwapPulse {
  0% {
    box-shadow: 0 0 20px $accent-light;
    transform: scale(1);
  }
  100% {
    box-shadow: 0 0 40px $accent-light, 0 0 60px $accent-blue;
    transform: scale(1.05);
  }
}
@keyframes swapPulse {
  0% {
    transform: scale(1);
  }
  30% {
    transform: scale(1.4);
    box-shadow: 0 0 70px $accent-light;
  }
  70% {
    transform: scale(0.95);
  }
  100% {
    transform: scale(1);
  }
}
@keyframes swapBackAnim {
  0% {
    transform: scale(1);
  }
  30% {
    transform: scale(0.7);
    opacity: 0.6;
    box-shadow: 0 0 50px $red-pupil;
  }
  70% {
    transform: scale(1.2);
    box-shadow: 0 0 60px $red-pupil;
  }
  100% {
    transform: scale(1);
  }
}
@keyframes eliminateAnim {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  40% {
    transform: scale(1.6);
    opacity: 1;
    box-shadow: 0 0 80px white;
  }
  70% {
    transform: scale(1.2);
    opacity: 0.7;
    box-shadow: 0 0 60px $accent-blue;
  }
  100% {
    transform: scale(0);
    opacity: 0;
  }
}
@keyframes appearAnim {
  0% {
    transform: scale(0);
    opacity: 0;
  }
  40% {
    transform: scale(1.4);
    opacity: 1;
    box-shadow: 0 0 70px $accent-blue;
  }
  70% {
    transform: scale(0.9);
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}
@keyframes skillHeal {
  0% {
    filter: brightness(1) drop-shadow(0 0 0 $accent-light);
  }
  25% {
    filter: brightness(1.8) drop-shadow(0 0 30px $accent-light);
  }
  50% {
    filter: brightness(1.4) drop-shadow(0 0 20px $accent-light);
  }
  75% {
    filter: brightness(1.6) drop-shadow(0 0 25px $accent-light);
  }
  100% {
    filter: brightness(1) drop-shadow(0 0 0 $accent-light);
  }
}

/* 死局提示 */
.deadlock-tip {
  position: fixed;
  top: 30%;
  left: 50%;
  transform: translateX(-50%);
  background: rgba($red-pupil, 0.9);
  color: white;
  font-size: 1.5rem;
  font-weight: bold;
  padding: 20px 40px;
  border-radius: 60px;
  border: 2px solid white;
  box-shadow: 0 0 40px $red-pupil;
  z-index: 3000;
  animation: tipPulse 0.5s infinite alternate;
  white-space: nowrap;
}
@keyframes tipPulse {
  0% {
    transform: translateX(-50%) scale(1);
    box-shadow: 0 0 40px $red-pupil;
  }
  100% {
    transform: translateX(-50%) scale(1.1);
    box-shadow: 0 0 80px $red-pupil;
  }
}

/* 角色技能栏 */
.character-skill-bar {
  position: relative;
  z-index: 10;
  display: flex;
  justify-content: center;
  gap: 15px;
  margin: 20px 0;
  padding: 15px;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 60px;
  border: 1px solid rgba($accent-blue, 0.3);
  backdrop-filter: blur(10px);
  flex-wrap: wrap;

  .skill-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 5px;
    animation: cursorAnimation_link 1s infinite step-start;
    opacity: 0.7;
    transition: all 0.3s;
    min-width: 70px;
    &.skill-ready {
      opacity: 1;
      .skill-avatar {
        animation: readyPulse 1s infinite;
        box-shadow: 0 0 30px $accent-light;
      }
    }
    &:hover {
      transform: translateY(-5px);
    }
  }
  .skill-icon-wrapper {
    position: relative;
    width: 60px;
    height: 60px;
    border-radius: 50%;
    overflow: hidden;
    border: 2px solid $accent-blue;
    .skill-avatar {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }

    .skill-count {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      color: white;
      font-size: 0.8rem;
      font-weight: bold;
      text-shadow: 0 0 5px black;
    }
  }
  .skill-name {
    font-size: 0.9rem;
    color: $accent-light;
  }
  .skill-effect {
    font-size: 0.7rem;
    color: rgba(255, 255, 255, 0.6);
    max-width: 80px;
    text-align: center;
    line-height: 1.2;
  }
  .skill-cooldown {
    font-size: 0.8rem;
    color: $red-pupil;
    font-weight: bold;
  }
}
@keyframes readyPulse {
  0% {
    box-shadow: 0 0 20px $accent-light;
  }
  50% {
    box-shadow: 0 0 50px $accent-light, 0 0 80px $accent-blue;
  }
  100% {
    box-shadow: 0 0 20px $accent-light;
  }
}

/* 技能特效文字 */
.skill-effect-text {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 2rem;
  font-weight: bold;
  color: white;
  text-shadow: 0 0 20px $accent-blue, 0 0 40px $accent-light;
  z-index: 2000;
  pointer-events: none;
  white-space: nowrap;
  background: rgba(0, 0, 0, 0.3);
  padding: 15px 30px;
  border-radius: 60px;
  border: 2px solid $accent-light;
  backdrop-filter: blur(10px);
}
.fade-scale-enter-active,
.fade-scale-leave-active {
  transition: all 0.3s ease;
}
.fade-scale-enter-from,
.fade-scale-leave-to {
  opacity: 0;
  transform: translate(-50%, -50%) scale(0.5);
}

.game-tip {
  position: relative;
  z-index: 10;
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.5);
  margin: 10px 0;
  text-align: center;
}

.submit-area {
  position: relative;
  z-index: 10;
  margin: 20px 0;
  .submit-btn {
    background: linear-gradient(135deg, $accent-blue, $accent-light);
    color: $bg-deep;
    border: none;
    padding: 12px 40px;
    border-radius: 40px;
    font-size: 1.2rem;
    font-weight: 600;
    animation: cursorAnimation_link 1s infinite step-start;
    box-shadow: 0 10px 20px rgba($accent-blue, 0.3);
    transition: 0.2s;
    &:hover:not(:disabled) {
      transform: translateY(-3px);
      box-shadow: 0 20px 30px rgba($accent-blue, 0.5);
    }
    &:disabled {
      opacity: 0.5;
      cursor: not-allowed;
    }
  }
}

/* 排行榜抽屉 */
.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s ease;
}
.slide-enter-from,
.slide-leave-to {
  transform: translateX(100%);
}

.ranking-drawer {
  position: fixed;
  top: 0;
  right: 0;
  width: 350px;
  height: 100vh;
  background: $bg-panel;
  border-left: 1px solid rgba($accent-blue, 0.3);
  box-shadow: -20px 0 40px rgba(0, 0, 0, 0.8);
  z-index: 1000;
  padding: 20px;
  overflow-y: auto;
  backdrop-filter: blur(12px);

  .drawer-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
    h3 {
      color: $accent-light;
      font-size: 1.4rem;
      margin: 0;
    }
    .close-btn {
      background: transparent;
      border: 1px solid rgba($accent-blue, 0.3);
      color: white;
      font-size: 1.2rem;
      width: 36px;
      height: 36px;
      border-radius: 50%;
      animation: cursorAnimation_link 1s infinite step-start;
      &:hover {
        background: rgba($accent-blue, 0.2);
      }
    }
  }

  .ranking-list {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }

  .ranking-item {
    display: flex;
    align-items: center;
    padding: 12px 16px;
    background: rgba(0, 0, 0, 0.3);
    border-radius: 40px;
    border: 1px solid rgba($accent-blue, 0.2);
    transition: 0.2s;

    .rank {
      width: 40px;
      font-weight: 700;
      color: $accent-light;
    }
    .name {
      flex: 1;
      font-weight: 500;
      color: white;
    }
    .score {
      font-weight: 700;
      color: $accent-light;
    }

    &.self-rank {
      background: rgba($accent-blue, 0.2);
      border-color: $accent-blue;
    }

    &:hover {
      transform: translateX(-4px);
      background: rgba($accent-blue, 0.15);
    }
  }

  .empty-rank {
    text-align: center;
    padding: 40px 0;
    color: rgba(255, 255, 255, 0.4);
  }
}

/* 移动端适配 */
@media (max-width: 600px) {
  .grid-cell {
    width: 45px;
    height: 45px;
  }
  .game-header {
    flex-direction: column;
    align-items: start;
    gap: 10px;
  }
  .character-skill-bar {
    gap: 8px;
    .skill-icon-wrapper {
      width: 45px;
      height: 45px;
    }
    .skill-name {
      font-size: 0.7rem;
    }
    .skill-effect {
      font-size: 0.6rem;
      max-width: 60px;
    }
  }
  .skill-effect-text {
    font-size: 1.5rem;
    padding: 10px 20px;
  }
  .deadlock-tip {
    font-size: 1rem;
    padding: 12px 24px;
  }
  .ranking-drawer {
    width: 90%;
  }
}
</style>
