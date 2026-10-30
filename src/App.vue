<script setup>
import { computed, onMounted, onUnmounted, ref } from 'vue'

const TASKS = [
  { id: 'dress', label: 'おきがえ', icon: '👗' },
  { id: 'breakfast', label: 'あさごはん', icon: '🍙' },
  { id: 'toilet', label: 'といれ', icon: '🚽' },
]
const STICKERS = ['🌸', '💖', '⭐', '🦄', '🐰', '🌈', '🎀', '🧸', '🍓', '🦋']
const KEY = 'oshitaku-sticker-v01'
const dayKey = () => {
  const now = new Date()
  return `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}-${String(now.getDate()).padStart(2, '0')}`
}
const emptyState = () => Object.fromEntries(TASKS.map(task => [task.id, null]))
const stickers = ref(emptyState())
const today = ref(dayKey())
const celebration = ref(false)
const burstId = ref(null)
const confetti = Array.from({ length: 22 }, (_, i) => ({
  left: `${(i * 37 + 11) % 100}%`,
  delay: `${(i % 7) * 0.09}s`,
  symbol: ['✨', '🌟', '💗', '🎉'][i % 4],
}))
let celebrationTimer
let burstTimer
let dayTimer

const completed = computed(() => TASKS.filter(task => Boolean(stickers.value[task.id])).length)
const allDone = computed(() => completed.value === TASKS.length)

function loadToday() {
  const currentDay = dayKey()
  today.value = currentDay
  try {
    const saved = JSON.parse(localStorage.getItem(KEY) || 'null')
    stickers.value = saved?.date === currentDay && saved?.stickers && typeof saved.stickers === 'object'
      ? Object.fromEntries(TASKS.map(task => [task.id, STICKERS.includes(saved.stickers[task.id]) ? saved.stickers[task.id] : null]))
      : emptyState()
  } catch {
    stickers.value = emptyState()
  }
  save()
}
function save() {
  try { localStorage.setItem(KEY, JSON.stringify({ date: today.value, stickers: stickers.value })) } catch { /* Private browsing or storage disabled */ }
}
function ensureToday() {
  if (today.value !== dayKey()) {
    clearTimeout(celebrationTimer)
    clearTimeout(burstTimer)
    celebration.value = false
    burstId.value = null
    loadToday()
  }
}
function addSticker(id) {
  ensureToday()
  // A completed task is intentionally locked to avoid accidental removal by children.
  if (stickers.value[id]) return
  const used = Object.values(stickers.value).filter(Boolean)
  const choices = STICKERS.filter(item => !used.includes(item))
  stickers.value = { ...stickers.value, [id]: choices[Math.floor(Math.random() * choices.length)] }
  save()
  burstId.value = id
  clearTimeout(burstTimer)
  burstTimer = setTimeout(() => { burstId.value = null }, 850)
  if (allDone.value) {
    celebration.value = true
    clearTimeout(celebrationTimer)
    celebrationTimer = setTimeout(() => { celebration.value = false }, 3800)
  }
}
function resetToday() {
  if (!window.confirm('きょうのシールを ぜんぶ はがしますか？')) return
  stickers.value = emptyState()
  celebration.value = false
  burstId.value = null
  clearTimeout(celebrationTimer)
  save()
}
function checkDay() { ensureToday() }
onMounted(() => {
  loadToday()
  dayTimer = setInterval(checkDay, 60_000)
  window.addEventListener('focus', checkDay)
  document.addEventListener('visibilitychange', checkDay)
})
onUnmounted(() => {
  clearInterval(dayTimer)
  clearTimeout(celebrationTimer)
  clearTimeout(burstTimer)
  window.removeEventListener('focus', checkDay)
  document.removeEventListener('visibilitychange', checkDay)
})
</script>

<template>
  <main class="page">
    <div class="decoration decoration-a" aria-hidden="true">✿</div>
    <div class="decoration decoration-b" aria-hidden="true">✦</div>
    <section class="board" aria-label="あさのおしたくシールちょう">
      <header class="header">
        <div class="eyebrow">✨ きょうも たのしく ✨</div>
        <h1>🌈 あさのおしたく 🌈</h1>
        <p>できたら タッチして シールを はろう！</p>
      </header>

      <div class="task-grid">
        <button v-for="task in TASKS" :key="task.id" class="task" :class="{ finished: stickers[task.id] }" type="button" :disabled="Boolean(stickers[task.id])" :aria-label="`${task.label} ${stickers[task.id] ? 'できた！' : 'シールをはる'}`" @click="addSticker(task.id)">
          <span class="task-icon" aria-hidden="true">{{ task.icon }}</span>
          <span class="task-label">{{ task.label }}</span>
          <span class="sticker-slot" :class="{ popping: burstId === task.id }" aria-hidden="true">
            <span v-if="stickers[task.id]" class="sticker">{{ stickers[task.id] }}</span>
            <span v-else class="plus">＋</span>
            <span v-if="burstId === task.id" class="sparkle sparkle-1">✦</span>
            <span v-if="burstId === task.id" class="sparkle sparkle-2">✧</span>
            <span v-if="burstId === task.id" class="sparkle sparkle-3">✦</span>
          </span>
          <span class="task-hint">{{ stickers[task.id] ? 'できたね！' : 'ここを タッチ！' }}</span>
        </button>
      </div>

      <footer class="footer">
        <div class="progress" aria-live="polite">{{ completed }} / {{ TASKS.length }} できたよ！</div>
        <div class="progress-stars" aria-hidden="true"><span v-for="n in TASKS.length" :key="n">{{ n <= completed ? '⭐' : '☆' }}</span></div>
        <div v-if="allDone" class="finished-message" role="status">🎉 やったー！ ぜんぶできた！ 🎉</div>
        <div v-else class="encourage">あと {{ TASKS.length - completed }} こ！</div>
      </footer>
    </section>

    <button class="parent-reset" type="button" @click="resetToday">おうちのひと：やりなおす</button>
    <div v-if="celebration" class="celebration" aria-hidden="true"><span v-for="(piece, index) in confetti" :key="index" class="confetti" :style="{ left: piece.left, animationDelay: piece.delay }">{{ piece.symbol }}</span></div>
  </main>
</template>
