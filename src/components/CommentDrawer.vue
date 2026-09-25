<script setup lang="ts">
import { computed, nextTick, onMounted, onBeforeUnmount, ref, watch } from 'vue'

type BrowserName = 'Chrome' | 'Edge' | 'Firefox' | 'Safari' | 'Opera' | '其他浏览器'
type PlatformName = 'Android' | 'iOS' | 'Windows' | 'Linux' | 'macOS' | '其他系统'

interface CommentRecord {
  id: string
  nickname: string
  content: string
  createdAt: number
  browser: BrowserName
  platform: PlatformName
}

const props = defineProps<{ open: boolean }>()
const emit = defineEmits<{ close: [] }>()

const storageKey = 'kami.comments.v1'
const nicknameKey = 'kami.comment.nickname.v1'
const comments = ref<CommentRecord[]>([])
const nickname = ref('')
const content = ref('')
const storageError = ref('')
const closeButton = ref<HTMLButtonElement | null>(null)
const commentCount = computed(() => comments.value.length)
const browsers: BrowserName[] = ['Chrome', 'Edge', 'Firefox', 'Safari', 'Opera', '其他浏览器']
const platforms: PlatformName[] = ['Android', 'iOS', 'Windows', 'Linux', 'macOS', '其他系统']

function getDevice(): { browser: BrowserName; platform: PlatformName } {
  const ua = navigator.userAgent
  let browser: BrowserName = '其他浏览器'
  if (/Edg\/|EdgiOS\//.test(ua)) browser = 'Edge'
  else if (/OPR\/|OPiOS\//.test(ua)) browser = 'Opera'
  else if (/Firefox\/|FxiOS\//.test(ua)) browser = 'Firefox'
  else if (/Chrome\/|CriOS\//.test(ua)) browser = 'Chrome'
  else if (/Safari\//.test(ua)) browser = 'Safari'

  let platform: PlatformName = '其他系统'
  if (/Android/.test(ua)) platform = 'Android'
  else if (/iPhone|iPad|iPod/.test(ua) || (/Macintosh/.test(ua) && navigator.maxTouchPoints > 1)) platform = 'iOS'
  else if (/Windows/.test(ua)) platform = 'Windows'
  else if (/Linux/.test(ua)) platform = 'Linux'
  else if (/Macintosh|Mac OS X/.test(ua)) platform = 'macOS'

  return { browser, platform }
}

function initial(name: string) {
  return Array.from(name.trim())[0]?.toLocaleUpperCase() || '?'
}

function dateLabel(timestamp: number) {
  return new Intl.DateTimeFormat('zh-CN', {
    year: 'numeric', month: '2-digit', day: '2-digit', hour: '2-digit', minute: '2-digit',
  }).format(timestamp)
}

function browserClass(browser: BrowserName) {
  return browser === '其他浏览器' ? 'browser-other' : `browser-${browser.toLowerCase()}`
}

function browserSymbol(browser: BrowserName) {
  return { Edge: 'e', Firefox: '✦', Safari: '⌖', Opera: 'O', Chrome: '', 其他浏览器: '◉' }[browser]
}

function loadComments() {
  try {
    nickname.value = localStorage.getItem(nicknameKey) || ''
    const saved = JSON.parse(localStorage.getItem(storageKey) || '[]') as unknown
    comments.value = Array.isArray(saved)
      ? saved.filter((item): item is CommentRecord =>
          !!item && typeof item === 'object' &&
          typeof item.id === 'string' && typeof item.nickname === 'string' &&
          typeof item.content === 'string' && typeof item.createdAt === 'number' &&
          Number.isFinite(item.createdAt) && item.createdAt > 0 &&
          browsers.includes(item.browser as BrowserName) && platforms.includes(item.platform as PlatformName),
        ).slice(0, 200)
      : []
    storageError.value = ''
  } catch {
    comments.value = []
    storageError.value = '浏览器存储不可用，评论暂时无法保存。'
  }
}

function submitComment() {
  const name = nickname.value.trim()
  const message = content.value.trim()
  if (!name || !message) return

  const comment: CommentRecord = {
    id: typeof crypto.randomUUID === 'function' ? crypto.randomUUID() : `${Date.now()}-${Math.random()}`,
    nickname: name,
    content: message,
    createdAt: Date.now(),
    ...getDevice(),
  }

  try {
    const next = [comment, ...comments.value].slice(0, 200)
    localStorage.setItem(nicknameKey, name)
    localStorage.setItem(storageKey, JSON.stringify(next))
    comments.value = next
    content.value = ''
    storageError.value = ''
  } catch {
    storageError.value = '保存失败，请检查浏览器是否允许本地存储。'
  }
}

function syncStorage(event: StorageEvent) {
  if (event.key === storageKey || event.key === nicknameKey) loadComments()
}

watch(() => props.open, async (open) => {
  if (open) {
    await nextTick()
    closeButton.value?.focus()
  }
})

onMounted(() => {
  loadComments()
  window.addEventListener('storage', syncStorage)
})
onBeforeUnmount(() => window.removeEventListener('storage', syncStorage))
</script>

<template>
  <Transition name="drawer-fade">
    <div v-if="open" class="comments-overlay" @keydown.esc="emit('close')">
      <button class="comments-backdrop" type="button" aria-label="关闭评论区" @click="emit('close')"></button>
      <aside class="comments-panel" role="dialog" aria-modal="true" aria-labelledby="comments-title">
        <header class="comments-header">
          <div>
            <span class="comments-eyebrow">GUESTBOOK</span>
            <h2 id="comments-title">评论 <span>{{ commentCount }}</span></h2>
          </div>
          <button ref="closeButton" class="comments-close" type="button" aria-label="关闭评论区" @click="emit('close')">
            <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M5 5l14 14M19 5 5 19" /></svg>
          </button>
        </header>

        <p class="comments-note">留一段话。评论仅存在当前浏览器，其他访客看不到。</p>

        <form class="comment-form" @submit.prevent="submitComment">
          <div class="comment-composer-heading">
            <span class="composer-avatar">{{ initial(nickname) }}</span>
            <label for="comment-nickname">你的昵称</label>
          </div>
          <input id="comment-nickname" v-model="nickname" type="text" maxlength="20" autocomplete="nickname" placeholder="怎么称呼你？" required />
          <label class="content-label" for="comment-content">想说的话</label>
          <textarea id="comment-content" v-model="content" maxlength="500" rows="4" placeholder="写下此刻的想法…" required></textarea>
          <div class="composer-footer">
            <span>{{ content.length }} / 500</span>
            <button type="submit" :disabled="!nickname.trim() || !content.trim()">发布评论 <span aria-hidden="true">↗</span></button>
          </div>
          <p v-if="storageError" class="storage-error" role="alert">{{ storageError }}</p>
        </form>

        <div class="comments-list-head"><span>留言</span><span>{{ commentCount }} 条</span></div>
        <div class="comments-feed">
          <div v-if="!comments.length" class="comments-empty">
            <span class="empty-sparkle" aria-hidden="true">✧</span>
            <p>还没有评论，写下第一句吧。</p>
          </div>
          <article v-for="comment in comments" :key="comment.id" class="comment-item">
            <span class="comment-avatar">{{ initial(comment.nickname) }}</span>
            <div class="comment-body">
              <div class="comment-byline"><strong>{{ comment.nickname }}</strong><time :datetime="new Date(comment.createdAt).toISOString()">{{ dateLabel(comment.createdAt) }}</time></div>
              <p>{{ comment.content }}</p>
              <div class="comment-device">
                <span class="platform-tag">{{ comment.platform }}</span>
                <span class="device-separator" aria-hidden="true">·</span>
                <span class="browser-tag">
                  <span class="browser-mark" :class="browserClass(comment.browser)" aria-hidden="true">{{ browserSymbol(comment.browser) }}</span>
                  {{ comment.browser }}
                </span>
              </div>
            </div>
          </article>
        </div>
      </aside>
    </div>
  </Transition>
</template>

<style scoped>
.comments-overlay { position: absolute; z-index: 20; inset: 0; }
.comments-backdrop { position: absolute; inset: 0; width: 100%; height: 100%; border: 0; padding: 0; background: rgba(7, 6, 17, .46); cursor: default; }
.comments-panel { position: absolute; top: 0; right: 0; bottom: 0; display: flex; flex-direction: column; width: min(430px, 100%); overflow: hidden; border-left: 1px solid rgba(255, 255, 255, .13); background: linear-gradient(150deg, rgba(36, 29, 54, .96), rgba(17, 15, 30, .97) 72%); box-shadow: -24px 0 70px rgba(0, 0, 0, .26); backdrop-filter: blur(28px); }
.comments-header { display: flex; align-items: center; justify-content: space-between; padding: 35px 32px 0; }
.comments-eyebrow { color: rgba(222, 208, 246, .5); font-size: 10px; font-weight: 600; letter-spacing: .3em; }
.comments-header h2 { margin: 7px 0 0; color: #faf7ff; font-family: 'Noto Serif SC', serif; font-size: 27px; font-weight: 400; letter-spacing: .08em; }
.comments-header h2 span { margin-left: 4px; color: rgba(239, 229, 255, .43); font-family: 'Manrope', sans-serif; font-size: 13px; font-weight: 500; vertical-align: middle; }
.comments-close { display: grid; width: 34px; height: 34px; place-items: center; border: 1px solid rgba(255, 255, 255, .14); border-radius: 50%; background: rgba(255, 255, 255, .04); color: rgba(255, 255, 255, .68); }
.comments-close:hover { background: rgba(255, 255, 255, .13); color: #fff; }
.comments-close svg { width: 14px; height: 14px; fill: none; stroke: currentColor; stroke-width: 1.5; stroke-linecap: round; }
.comments-note { margin: 15px 32px 25px; color: rgba(238, 229, 247, .53); font-size: 11px; line-height: 1.7; letter-spacing: .025em; }
.comment-form { margin: 0 32px; border: 1px solid rgba(255, 255, 255, .12); border-radius: 16px; padding: 17px; background: rgba(255, 255, 255, .045); }
.comment-composer-heading { display: flex; align-items: center; gap: 10px; margin-bottom: 12px; }
.composer-avatar, .comment-avatar { display: grid; flex: 0 0 auto; place-items: center; border: 1px solid rgba(255, 255, 255, .3); border-radius: 50%; background: linear-gradient(145deg, rgba(234, 218, 255, .2), rgba(138, 121, 185, .1)); color: #fff; font-family: 'Noto Serif SC', serif; font-weight: 500; }
.composer-avatar { width: 29px; height: 29px; font-size: 13px; }
.comment-form label { color: rgba(255, 255, 255, .83); font-size: 12px; letter-spacing: .055em; }
.comment-form input, .comment-form textarea { display: block; width: 100%; border: 1px solid rgba(255, 255, 255, .12); border-radius: 8px; outline: 0; padding: 9px 11px; background: rgba(5, 4, 15, .2); color: #fff; font: 12px/1.6 'Manrope', 'Noto Sans SC', sans-serif; transition: border-color .2s, background .2s; }
.comment-form input:focus, .comment-form textarea:focus { border-color: rgba(220, 199, 255, .58); background: rgba(5, 4, 15, .34); }
.comment-form input::placeholder, .comment-form textarea::placeholder { color: rgba(235, 226, 247, .34); }
.comment-form textarea { min-height: 88px; max-height: 160px; resize: vertical; }
.content-label { display: block; margin: 14px 0 8px; }
.composer-footer { display: flex; align-items: center; justify-content: space-between; gap: 12px; margin-top: 12px; }
.composer-footer > span { color: rgba(245, 236, 255, .35); font-size: 10px; }
.composer-footer button { border: 1px solid rgba(237, 222, 255, .46); border-radius: 100px; padding: 7px 13px; background: rgba(231, 207, 255, .13); color: #f6efff; font-size: 11px; letter-spacing: .06em; transition: background .2s, border-color .2s; }
.composer-footer button:not(:disabled):hover { border-color: rgba(237, 222, 255, .8); background: rgba(231, 207, 255, .25); }
.composer-footer button:disabled { cursor: not-allowed; opacity: .42; }
.composer-footer button span { margin-left: 6px; font-size: 14px; }
.storage-error { margin: 10px 0 0; color: #ffd1d1; font-size: 11px; }
.comments-list-head { display: flex; justify-content: space-between; margin: 27px 32px 0; padding-bottom: 10px; border-bottom: 1px solid rgba(255, 255, 255, .12); color: rgba(255, 255, 255, .75); font-size: 11px; letter-spacing: .08em; }
.comments-list-head span:last-child { color: rgba(255, 255, 255, .4); }
.comments-feed { flex: 1; min-height: 0; overflow-y: auto; overscroll-behavior: contain; scrollbar-color: rgba(220, 204, 247, .22) transparent; padding: 0 32px 30px; }
.comments-empty { padding: 42px 0; text-align: center; color: rgba(239, 229, 255, .47); }
.empty-sparkle { color: rgba(233, 216, 255, .65); font-size: 29px; }
.comments-empty p { margin: 8px 0; font-size: 12px; }
.comment-item { display: flex; gap: 12px; padding: 20px 0; border-bottom: 1px solid rgba(255, 255, 255, .09); }
.comment-avatar { width: 35px; height: 35px; font-size: 15px; }
.comment-body { min-width: 0; flex: 1; }
.comment-byline { display: flex; flex-wrap: wrap; align-items: baseline; justify-content: space-between; gap: 5px 12px; }
.comment-byline strong { color: #f8f2ff; font-size: 12px; font-weight: 600; overflow-wrap: anywhere; }
.comment-byline time { color: rgba(245, 234, 255, .38); font-size: 10px; }
.comment-body p { margin: 10px 0 12px; color: rgba(255, 251, 255, .78); font-size: 12px; line-height: 1.85; overflow-wrap: anywhere; white-space: pre-wrap; }
.comment-device { display: flex; align-items: center; gap: 6px; color: rgba(230, 218, 247, .47); font-size: 10px; }
.browser-tag { display: inline-flex; align-items: center; gap: 5px; }
.browser-mark { display: grid; width: 13px; height: 13px; place-items: center; border-radius: 50%; font: 700 10px/1 Arial, sans-serif; }
.browser-chrome { border: 3px solid #e6b84b; background: #4a9fe3; box-shadow: 2px 2px 0 #68ac74, -2px -1px 0 #d9706d; }
.browser-edge { background: linear-gradient(135deg, #31c99f, #266cc3); color: white; font-style: italic; }
.browser-firefox { background: linear-gradient(135deg, #f3b84d, #b65add); color: white; }
.browser-safari { background: #65a7df; color: #fff; }
.browser-opera { background: #da5671; color: #fff; }
.browser-other { background: #9c91af; color: #fff; }
.drawer-fade-enter-active, .drawer-fade-leave-active { transition: opacity .25s ease; }
.drawer-fade-enter-active .comments-panel, .drawer-fade-leave-active .comments-panel { transition: transform .32s cubic-bezier(.22, .61, .36, 1); }
.drawer-fade-enter-from, .drawer-fade-leave-to { opacity: 0; }
.drawer-fade-enter-from .comments-panel, .drawer-fade-leave-to .comments-panel { transform: translateX(30px); }
@media (max-width: 700px) {
  .comments-panel { width: 100%; border-left: 0; }
  .comments-header { padding: 22px 22px 0; }
  .comments-note { margin: 12px 22px 18px; }
  .comment-form { margin: 0 22px; }
  .comments-list-head { margin: 21px 22px 0; }
  .comments-feed { padding: 0 22px 20px; }
}
@media (max-height: 620px) {
  .comments-panel { overflow-y: auto; }
  .comments-feed { flex: none; min-height: 150px; overflow: visible; }
}
</style>
