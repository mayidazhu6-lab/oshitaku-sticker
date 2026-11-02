<script setup>
import { computed, onMounted, onUnmounted, ref } from 'vue'

const TASKS = [
  { id: 'dress', label: 'おきがえ', icon: '👗' },
  { id: 'breakfast', label: 'あさごはん', icon: '🍙' },
  { id: 'toilet', label: 'といれ', icon: '🚽' },
]
const STICKERS = ['🌸', '💖', '⭐', '🦄', '🐰', '🌈', '🎀', '🧸', '🍓', '🦋']
const KEY = 'oshitaku-sticker-v01'
const TOILET_KEY = 'oshitaku-toilet-adventure-v01'
const MAPS = [
  { name: 'おはなばたけ', icon: '🌷', scene: '🌼 🌸 🦋 🌻 🌷', color: 'flower' },
  { name: 'どうぶつのもり', icon: '🌳', scene: '🌲 🐰 🍄 🦊 🌿', color: 'forest' },
  { name: 'にじのそら', icon: '🌈', scene: '☁️ ⭐ 🌈 🕊️ ☀️', color: 'sky' },
  { name: 'おしろのこみち', icon: '🏰', scene: '🌹 🏰 💐 🦄 ✨', color: 'castle' },
  { name: 'うみのぼうけん', icon: '🐬', scene: '🐚 🐠 🐬 🪸 🐢', color: 'sea' },
]
const page = ref('home')
const menuOpen = ref(false)
const showHistory = ref(false)
const historyMap = ref(0)
const morningNotice = ref(false)
const toiletCount = ref(0)
const displayedCount = ref(0)
const adventureFromMorning = ref(false)
const adventureCelebration = ref(false)
const adventureBusy = ref(false)
const mapIndex = computed(() => Math.floor(displayedCount.value / 10))
const mapProgress = computed(() => displayedCount.value % 10)
const isCompletingMap = computed(() => adventureCelebration.value && toiletCount.value % 10 === 0)
const map = computed(() => MAPS[mapIndex.value % MAPS.length])
const pathCells = [
  { x: 9, y: 73 }, { x: 27, y: 75 }, { x: 45, y: 75 },
  { x: 64, y: 75 }, { x: 82, y: 68 }, { x: 83, y: 40 },
  { x: 65, y: 27 }, { x: 47, y: 27 }, { x: 29, y: 27 }, { x: 11, y: 27 },
]
let adventureTimer
let transitionTimer
function loadToilet() {
  try {
    const value = JSON.parse(localStorage.getItem(TOILET_KEY) || 'null')
    toiletCount.value = Number.isSafeInteger(value?.count) && value.count >= 0 ? value.count : 0
  } catch { toiletCount.value = 0 }
  displayedCount.value = toiletCount.value
}
function saveToilet() {
  try { localStorage.setItem(TOILET_KEY, JSON.stringify({ count: toiletCount.value })) } catch { /* storage unavailable */ }
}
function goTo(destination) {
  clearTimeout(transitionTimer)
  clearTimeout(adventureTimer)
  adventureBusy.value = false
  adventureCelebration.value = false
  adventureFromMorning.value = false
  showHistory.value = false
  morningNotice.value = false
  displayedCount.value = toiletCount.value
  menuOpen.value = false
  page.value = destination
}
function enterAdventureFromMorning() {
  morningNotice.value = false
  adventureFromMorning.value = true
  page.value = 'adventure'
  menuOpen.value = false
}
function addAdventureSticker() {
  if (adventureBusy.value || adventureCelebration.value || showHistory.value) return
  adventureBusy.value = true
  toiletCount.value += 1
  saveToilet()
  const finished = toiletCount.value % 10 === 0
  if (finished) adventureCelebration.value = true
  else displayedCount.value = toiletCount.value
  adventureTimer = setTimeout(() => {
    adventureCelebration.value = false
    displayedCount.value = toiletCount.value
    adventureBusy.value = false
    if (adventureFromMorning.value) {
      adventureFromMorning.value = false
      page.value = 'morning'
    }
  }, finished ? 2700 : 900)
}
function undoToilet() {
  if (adventureBusy.value || toiletCount.value === 0) return
  if (!window.confirm('さいごのシールを 1まい もどしますか？')) return
  toiletCount.value -= 1
  displayedCount.value = toiletCount.value
  saveToilet()
}


const TREE_KEY = 'oshitaku-dekita-tree-v01'
const treeCount = ref(0)
const treeGrowing = ref(false)
const treeMilestones = [
  { height: 1, icon: '🧒', label: 'こども' },
  { height: 2, icon: '🚪', label: 'ドア' },
  { height: 5, icon: '🦒', label: 'キリン' },
  { height: 10, icon: '🏠', label: 'おうち' },
  { height: 30, icon: '🏢', label: 'ビル' },
  { height: 100, icon: '🏙️', label: 'たかいビル' },
  { height: 333, icon: '🗼', label: 'とうきょうタワー' },
  { height: 634, icon: '🗼', label: 'スカイツリー' },
  { height: 10000, icon: '✈️', label: 'ひこうき' },
]
const treeNearby = computed(() => [...treeMilestones].reverse().find(item => treeCount.value >= item.height) || null)
const treeNext = computed(() => treeMilestones.find(item => treeCount.value < item.height) || null)
const treeSize = computed(() => Math.min(1.16, 0.48 + Math.log2(treeCount.value + 1) * 0.115))
let treeTimer
function loadTree() {
  try {
    const value = JSON.parse(localStorage.getItem(TREE_KEY) || 'null')
    treeCount.value = Number.isSafeInteger(value?.count) && value.count >= 0 ? value.count : 0
  } catch { treeCount.value = 0 }
}
function saveTree() {
  try { localStorage.setItem(TREE_KEY, JSON.stringify({ count: treeCount.value })) } catch { /* storage unavailable */ }
}
function growTree() {
  if (treeGrowing.value || treeCount.value >= Number.MAX_SAFE_INTEGER) return
  treeCount.value += 1
  saveTree()
  treeGrowing.value = true
  clearTimeout(treeTimer)
  treeTimer = setTimeout(() => { treeGrowing.value = false }, 650)
}
function undoTree() {
  if (treeGrowing.value || treeCount.value === 0) return
  if (!window.confirm('さいごの「できた！」を 1かい もどしますか？')) return
  treeCount.value -= 1
  saveTree()
}

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
  if (id === 'toilet' && page.value === 'morning') {
    morningNotice.value = true
    transitionTimer = setTimeout(enterAdventureFromMorning, 1500)
  }
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
  loadToilet()
  loadTree()
  dayTimer = setInterval(checkDay, 60_000)
  window.addEventListener('focus', checkDay)
  document.addEventListener('visibilitychange', checkDay)
})
onUnmounted(() => {
  clearInterval(dayTimer)
  clearTimeout(treeTimer)
  clearTimeout(adventureTimer)
  clearTimeout(transitionTimer)
  clearTimeout(celebrationTimer)
  clearTimeout(burstTimer)
  window.removeEventListener('focus', checkDay)
  document.removeEventListener('visibilitychange', checkDay)
})
</script>

<template>
  <main class="page" :class="{ 'adventure-page': page === 'adventure', 'tree-page': page === 'tree' }">
    <button v-if="page !== 'home'" class="menu-toggle" type="button" aria-label="メニューをひらく" :aria-expanded="menuOpen" @click="menuOpen = !menuOpen">☰</button>
    <nav v-if="menuOpen && page !== 'home'" class="mode-menu" aria-label="モードきりかえ">
      <button type="button" @click="goTo('home')">🏠 ホーム</button>
      <button type="button" @click="goTo('morning')">☀️ あさのおしたく</button>
      <button type="button" @click="goTo('adventure')">🗺️ トイレのぼうけん</button>
      <button type="button" @click="goTo('tree')">🌳 できたのき</button>
    </nav>
    <section v-if="page === 'home'" class="board home-board">
      <header class="header"><div class="eyebrow">✨ おしたくシール ✨</div><h1>きょうは なにをする？</h1></header>
      <div class="home-choices">
        <button class="home-choice morning-choice" type="button" @click="goTo('morning')"><span>☀️👗</span><strong>あさのおしたく</strong></button>
        <button class="home-choice adventure-choice" type="button" @click="goTo('adventure')"><span>🚽🗺️</span><strong>トイレのぼうけん</strong></button>
        <button class="home-choice tree-choice" type="button" @click="goTo('tree')"><span>🌱🌳</span><strong>できたのき</strong></button>
      </div>
    </section>

    <section v-if="page === 'tree'" class="board tree-board">
      <header class="header tree-header">
        <div class="eyebrow">🌱 ちいさな できたが おおきな きに 🌱</div>
        <h1>🌳 できたのき</h1>
      </header>
      <div class="tree-scene" role="img" :aria-label="treeCount === 0 ? 'ふたばが はえているよ' : treeCount + 'メートルに そだった き'">
        <div class="tree-sun" aria-hidden="true">☀️</div>
        <div class="tree-cloud tree-cloud-a" aria-hidden="true">☁️</div>
        <div class="tree-cloud tree-cloud-b" aria-hidden="true">☁️</div>
        <div class="tree-ground" aria-hidden="true"></div>
        <svg class="tree-art" :class="{ 'tree-pop': treeGrowing }" viewBox="0 0 320 290" aria-hidden="true">
          <g v-if="treeCount === 0">
            <path d="M160 263 Q159 234 160 215" fill="none" stroke="#67A764" stroke-width="9" stroke-linecap="round"/>
            <ellipse cx="139" cy="219" rx="27" ry="13" fill="#86CB83" transform="rotate(25 139 219)"/>
            <ellipse cx="182" cy="207" rx="27" ry="13" fill="#74BB72" transform="rotate(-28 182 207)"/>
          </g>
          <g v-else :style="{ transform: 'translateY(265px) scale(' + treeSize + ') translateY(-265px)' }" class="tree-shape">
            <path d="M144 266 Q149 208 143 160 L176 160 Q168 210 177 266 Z" fill="#A97952"/>
            <path d="M157 221 Q146 185 120 169 M166 201 Q185 175 205 166" fill="none" stroke="#A97952" stroke-width="15" stroke-linecap="round"/>
            <circle cx="159" cy="121" r="68" fill="#77BD78"/>
            <circle cx="115" cy="155" r="47" fill="#8FCE85"/>
            <circle cx="206" cy="153" r="48" fill="#81C783"/>
            <circle cx="145" cy="90" r="48" fill="#9AD990"/>
            <circle cx="192" cy="105" r="45" fill="#91D18A"/>
            <circle cx="131" cy="134" r="23" fill="#A5DD99" opacity=".7"/>
          </g>
        </svg>
        <div v-if="treeNearby" class="tree-comparison" aria-hidden="true"><span>{{ treeNearby.icon }}</span><small>{{ treeNearby.label }}</small></div>
        <div v-if="treeGrowing" class="tree-sparkle" aria-hidden="true">✨ ぐんっ！ ✨</div>
      </div>
      <div class="tree-stats" aria-live="polite">
        <strong>{{ treeCount === 0 ? '0 m' : treeCount.toLocaleString('ja-JP') + ' m' }}</strong>
        <span>{{ treeCount === 0 ? 'まだ ふたばだよ！' : 'きの たかさ' }}</span>
        <p v-if="treeNext">つぎは {{ treeNext.icon }} {{ treeNext.label }}！ あと {{ (treeNext.height - treeCount).toLocaleString('ja-JP') }} m</p>
        <p v-else>そらの ずっと うえまで のびていくよ！</p>
      </div>
      <button type="button" class="tree-grow-button" :disabled="treeGrowing" @click="growTree">⭐ できた！ ⭐</button>
      <button type="button" class="tree-undo" :disabled="treeGrowing || treeCount === 0" @click="undoTree">おうちのひと：1かい もどす</button>
    </section>
    <section v-if="page === 'adventure'" class="board adventure-board" :class="'map-' + map.color">
      <header class="header">
        <div class="eyebrow">🗺️ トイレのぼうけん 🗺️</div>
        <h1>{{ map.icon }} {{ map.name }}</h1>
        <p>だい {{ mapIndex + 1 }} マップ ・ {{ mapProgress }} / 10 まい</p>
      </header>
      <div v-if="showHistory" class="history-panel"><h2>📖 ぼうけんのきろく</h2><p>これまでの マップを みてみよう！</p><div class="history-list"><button v-for="n in Math.floor(toiletCount / 10)" :key="n" type="button" @click="historyMap = n - 1">{{ MAPS[(n - 1) % MAPS.length].icon }} {{ n }} ばんめの マップ</button></div><div v-if="historyMap < Math.floor(toiletCount / 10)" class="history-preview">{{ MAPS[historyMap % MAPS.length].scene }}<p>⭐ {{ historyMap + 1 }} ばんめの マップ クリア！</p></div><button type="button" class="history-back" @click="showHistory = false">ぼうけんに もどる</button></div>
      <div v-else class="adventure-map">
        <div class="map-scenery" aria-hidden="true">{{ map.scene }}</div>
        <svg class="map-path" viewBox="0 0 100 100" preserveAspectRatio="none" aria-hidden="true">
          <polyline :points="pathCells.map(p => p.x + ',' + p.y).join(' ')" fill="none" stroke="#fff" stroke-width="5" stroke-linecap="round" stroke-linejoin="round" stroke-dasharray="2 2"/>
        </svg>
        <button v-for="(cell, index) in pathCells" :key="index" type="button" class="map-cell"
          :class="{ filled: index < mapProgress || (isCompletingMap && index === 9), next: index === mapProgress && !isCompletingMap }"
          :style="{ left: cell.x + '%', top: cell.y + '%' }"
          :disabled="index !== mapProgress || adventureBusy"
          :aria-label="index < mapProgress || (isCompletingMap && index === 9) ? (index + 1) + 'まい はったよ' : index === mapProgress ? 'つぎのシールをはる' : (index + 1) + 'ばん'"
          @click="addAdventureSticker">
          {{ index < mapProgress || (isCompletingMap && index === 9) ? STICKERS[(mapIndex * 10 + index) % STICKERS.length] : index === mapProgress ? '＋' : index + 1 }}
        </button>
      </div>
      <footer v-if="!showHistory" class="adventure-footer">
        <strong>⭐ ぜんぶで {{ toiletCount }} まい！</strong>
        <span>あと {{ 10 - mapProgress }} まいで キラキラ！</span>
        <button class="undo-toilet" type="button" @click="showHistory = true">📖 ぼうけんのきろく</button>
        <button class="undo-toilet" type="button" @click="undoToilet" :disabled="adventureBusy || toiletCount === 0">おうちのひと：1まい もどす</button>
      </footer>
      <div v-if="adventureCelebration" class="adventure-overlay" role="status">
        <div class="adventure-congrats">✨🎉✨<br>10かい たっせい！<br>やったー！<br>✨🌈✨</div>
      </div>
    </section>
    <template v-if="page === 'morning'">
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
    <div v-if="morningNotice" class="morning-notice" role="status">✨ トイレのぼうけんにも<br>シールを はろう！ ✨</div>
    </template>
    <div v-if="page === 'morning' && celebration" class="celebration" aria-hidden="true"><span v-for="(piece, index) in confetti" :key="index" class="confetti" :style="{ left: piece.left, animationDelay: piece.delay }">{{ piece.symbol }}</span></div>
  </main>
</template>
