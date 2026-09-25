<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref } from 'vue'

const homeHref = `${import.meta.env.BASE_URL}`
const commentMount = ref<HTMLElement | null>(null)
const loadState = ref<'loading' | 'ready' | 'error'>('loading')
const copyStatus = ref('')
let observer: MutationObserver | undefined

type PlatformName = 'Android' | 'iOS' | 'Windows' | 'Linux' | 'macOS' | '其他系统'
type BrowserName = 'Chrome' | 'Edge' | 'Firefox' | 'Safari' | 'Opera' | '其他浏览器'

function detectCommentSource() {
  const ua = navigator.userAgent
  let browser: BrowserName = '其他浏览器'
  let browserSlug = 'other'
  let browserSymbol = '◉'

  if (/Edg\/|EdgiOS\/|EdgA\//.test(ua)) {
    browser = 'Edge'
    browserSlug = 'edge'
    browserSymbol = 'e'
  } else if (/OPR\/|OPiOS\//.test(ua)) {
    browser = 'Opera'
    browserSlug = 'opera'
    browserSymbol = 'O'
  } else if (/Firefox\/|FxiOS\//.test(ua)) {
    browser = 'Firefox'
    browserSlug = 'firefox'
    browserSymbol = '✦'
  } else if (/Chrome\/|CriOS\//.test(ua)) {
    browser = 'Chrome'
    browserSlug = 'chrome'
    browserSymbol = ''
  } else if (/Safari\//.test(ua)) {
    browser = 'Safari'
    browserSlug = 'safari'
    browserSymbol = '⌖'
  }

  let platform: PlatformName = '其他系统'
  if (/Android/i.test(ua)) platform = 'Android'
  else if (/iPhone|iPad|iPod/i.test(ua) || (/Macintosh/i.test(ua) && navigator.maxTouchPoints > 1)) platform = 'iOS'
  else if (/Windows/i.test(ua)) platform = 'Windows'
  else if (/Linux/i.test(ua)) platform = 'Linux'
  else if (/Macintosh|Mac OS X/i.test(ua)) platform = 'macOS'

  return { platform, browser, browserSlug, browserSymbol }
}

const commentSource = detectCommentSource()
const platformGlyph: Record<PlatformName, string> = {
  Android: '🤖', iOS: '📱', Windows: '🪟', Linux: '🐧', macOS: '🍎', '其他系统': '💻',
}
const browserGlyph: Record<BrowserName, string> = {
  Chrome: '🌐', Edge: '🔷', Firefox: '🦊', Safari: '🧭', Opera: '🔴', '其他浏览器': '🌍',
}
const commentSourceText = `${platformGlyph[commentSource.platform]} ${commentSource.platform} · ${browserGlyph[commentSource.browser]} ${commentSource.browser}`

async function copyCommentSource() {
  try {
    await navigator.clipboard.writeText(commentSourceText)
    copyStatus.value = '已复制；粘贴到评论里即可'
  } catch {
    copyStatus.value = '复制不可用，请手动选择标签文本'
  }
}

onMounted(() => {
  const mount = commentMount.value
  if (!mount) return

  observer = new MutationObserver(() => {
    if (mount.querySelector('iframe.giscus-frame')) loadState.value = 'ready'
  })
  observer.observe(mount, { childList: true, subtree: true })

  const script = document.createElement('script')
  script.src = 'https://giscus.app/client.js'
  script.async = true
  script.crossOrigin = 'anonymous'
  script.dataset.repo = 'Neon0815/blog'
  script.dataset.repoId = 'R_kgDOUq6HFA'
  script.dataset.category = 'Announcements'
  script.dataset.categoryId = 'DIC_kwDOUq6HFM4DGXOU'
  script.dataset.mapping = 'pathname'
  script.dataset.strict = '1'
  script.dataset.reactionsEnabled = '1'
  script.dataset.emitMetadata = '0'
  script.dataset.inputPosition = 'top'
  script.dataset.theme = 'dark_dimmed'
  script.dataset.lang = 'zh-CN'
  script.addEventListener('error', () => { loadState.value = 'error' }, { once: true })
  mount.append(script)
})

onBeforeUnmount(() => observer?.disconnect())
</script>

<template>
  <main class="comments-page">
    <div class="comments-page-backdrop" :style="{ backgroundImage: `url(${homeHref}backgrounds/bg1.png)` }" aria-hidden="true"></div>
    <div class="comments-page-shade" aria-hidden="true"></div>
    <div class="comments-page-shell">
      <header class="comments-page-header">
        <a class="comments-home-link" :href="homeHref"><span aria-hidden="true">←</span> 回到主页</a>
        <span class="comments-page-mark">KAMI / GUESTBOOK</span>
      </header>

      <section class="comments-page-content">
        <div class="comments-intro">
          <span class="comments-kicker">A LITTLE NOTE, LEFT HERE</span>
          <h1>把想说的话，<br />留在这里。</h1>
          <p>每一段留言都会保存在公开的讨论区，来访的人都能读到，也能加入对话。</p>
        </div>

        <section class="shared-comments" aria-label="公开评论">
          <div class="shared-comments-heading">
            <div><span>OPEN DISCUSSION</span><h2>留言</h2></div>
            <span class="shared-comments-status"><i></i> 公共留言</span>
          </div>
          <div class="comment-source-card">
            <div class="comment-source-copy">
              <span class="comment-source-title">本设备来源 <span>可选</span></span>
              <div class="comment-source-tags">
                <span class="comment-source-tag">
                  <svg v-if="commentSource.platform === 'Windows'" viewBox="0 0 24 24" aria-hidden="true"><path d="M3 5.2 10.4 4v7H3zm8.7-1.4L21 2v9h-9.3zM3 12.8h7.4v7L3 18.6zm8.7 0H21v9l-9.3-1.8z" /></svg>
                  <svg v-else-if="commentSource.platform === 'Android'" viewBox="0 0 24 24" aria-hidden="true"><path d="M7 8a5 5 0 0 1 10 0H7Zm-1.5 1.2h13v7.1a1.5 1.5 0 0 1-1.5 1.5H7a1.5 1.5 0 0 1-1.5-1.5zM5.5 11H4v4h1.5zm14.5 0h-1.5v4H20zM8 17.8v2m8-2v2M8.2 4.4 7.1 2.8m8.7 1.6 1.1-1.6" /><circle cx="9.5" cy="6.2" r=".55" class="icon-cutout" /><circle cx="14.5" cy="6.2" r=".55" class="icon-cutout" /></svg>
                  <svg v-else-if="commentSource.platform === 'iOS'" viewBox="0 0 24 24" aria-hidden="true"><rect x="6.5" y="2.5" width="11" height="19" rx="2.2" /><path d="M10 5h4m-3 13.5h2" /></svg>
                  <svg v-else-if="commentSource.platform === 'Linux'" viewBox="0 0 24 24" aria-hidden="true"><path d="M8 19.5c-2.2.7-4 .1-4-1.2 0-.8.9-1.3 2.2-1.6-.6-1.5-.9-3.4-.5-5.4C6.4 8 8.6 5 12 5s5.6 3 6.3 6.3c.4 2-.1 4.2-.9 5.6 1.5.3 2.6.9 2.6 1.7 0 1.3-2.1 1.8-4.4.9l-1.2-1.2h-4.9z" /><path d="M8 9.5 6.5 6.8m8.7 1.4 2.2-2.4" /><circle cx="10" cy="11" r=".7" class="icon-cutout" /><circle cx="14" cy="11" r=".7" class="icon-cutout" /><path d="m10 14 2 1 2-1" /></svg>
                  <svg v-else viewBox="0 0 24 24" aria-hidden="true"><rect x="3" y="3.5" width="18" height="13" rx="1.8" /><path d="M8 20.5h8m-4-4v4" /></svg>
                  {{ commentSource.platform }}
                </span>
                <span class="comment-source-divider" aria-hidden="true">·</span>
                <span class="comment-source-tag">
                  <span class="source-browser-mark" :class="`browser-${commentSource.browserSlug}`" aria-hidden="true">{{ commentSource.browserSymbol }}</span>
                  {{ commentSource.browser }}
                </span>
              </div>
            </div>
            <button class="comment-source-button" type="button" @click="copyCommentSource">
              <svg viewBox="0 0 24 24" aria-hidden="true"><rect x="8" y="8" width="12" height="13" rx="2" /><path d="M16 8V5a2 2 0 0 0-2-2H6a2 2 0 0 0-2 2v10a2 2 0 0 0 2 2h2" /></svg>
              复制来源标签
            </button>
            <span v-if="copyStatus" class="comment-source-feedback" role="status">{{ copyStatus }}</span>
            <p>复制后粘贴到评论里；只有主动发布，设备信息才会公开。</p>
          </div>
          <p v-if="loadState === 'loading'" class="giscus-loading">正在连接留言区…</p>
          <p v-if="loadState === 'error'" class="giscus-error">留言区加载失败，请稍后刷新重试。</p>
          <div ref="commentMount" class="giscus"></div>
          <p class="shared-comments-footnote">发布留言需要使用 GitHub 账号授权；你的 GitHub 昵称和头像会显示在留言旁。</p>
        </section>
      </section>

      <footer class="comments-page-footer">KAMI <span>·</span> SOME THINGS DESERVE TO BE SHARED</footer>
    </div>
  </main>
</template>

<style scoped>
:global(html), :global(body), :global(#app) { width: 100%; height: auto; min-height: 100%; overflow-x: hidden; overflow-y: auto; }
:global(body) { background: #0b0917; }
.comments-page { position: relative; min-height: 100svh; isolation: isolate; color: #f7f2ff; background: #0d0b17; }
.comments-page-backdrop, .comments-page-shade { position: fixed; z-index: -2; inset: 0; pointer-events: none; }
.comments-page-backdrop { background-position: center; background-size: cover; filter: brightness(.22) saturate(.65); transform: scale(1.03); }
.comments-page-shade { z-index: -1; background: linear-gradient(90deg, rgba(12, 10, 22, .94), rgba(12, 10, 22, .78) 55%, rgba(12, 10, 22, .88)), linear-gradient(180deg, rgba(9, 7, 18, .22), rgba(9, 7, 18, .86)); }
.comments-page-shell { width: min(1160px, 100% - 64px); min-height: 100svh; margin: 0 auto; display: flex; flex-direction: column; }
.comments-page-header { display: flex; align-items: center; justify-content: space-between; padding: 35px 0; border-bottom: 1px solid rgba(255, 255, 255, .12); }
.comments-home-link { display: inline-flex; align-items: center; gap: 9px; color: rgba(255, 255, 255, .7); font-size: 12px; letter-spacing: .08em; text-decoration: none; transition: color .2s; }
.comments-home-link:hover { color: #fff; }
.comments-home-link span { font-size: 17px; }
.comments-page-mark { color: rgba(244, 232, 255, .4); font-size: 10px; letter-spacing: .24em; }
.comments-page-content { display: grid; grid-template-columns: minmax(230px, .72fr) minmax(0, 1.28fr); gap: clamp(44px, 8vw, 112px); flex: 1; align-items: start; padding: clamp(72px, 11vh, 125px) 0 85px; }
.comments-intro { position: sticky; top: 72px; padding-top: 8px; }
.comments-kicker, .shared-comments-heading > div > span { color: rgba(222, 207, 244, .47); font-size: 9px; font-weight: 600; letter-spacing: .28em; }
.comments-intro h1 { margin: 25px 0 22px; color: rgba(255, 252, 255, .94); font-family: 'Noto Serif SC', serif; font-size: clamp(29px, 3.2vw, 43px); font-weight: 400; letter-spacing: .12em; line-height: 1.75; }
.comments-intro p { max-width: 290px; margin: 0; color: rgba(243, 235, 251, .56); font-size: 12px; line-height: 2; letter-spacing: .045em; }
.shared-comments { min-width: 0; border: 1px solid rgba(255, 255, 255, .14); border-radius: 17px; padding: 25px 26px 18px; background: rgba(22, 19, 35, .58); box-shadow: 0 22px 65px rgba(0, 0, 0, .2); backdrop-filter: blur(20px); }
.shared-comments-heading { display: flex; align-items: center; justify-content: space-between; gap: 16px; padding-bottom: 19px; border-bottom: 1px solid rgba(255, 255, 255, .11); }
.shared-comments-heading h2 { margin: 7px 0 0; color: rgba(255, 255, 255, .94); font-family: 'Noto Serif SC', serif; font-size: 21px; font-weight: 400; letter-spacing: .12em; }
.shared-comments-status { display: inline-flex; align-items: center; gap: 7px; color: rgba(231, 222, 241, .53); font-size: 10px; white-space: nowrap; }
.shared-comments-status i { width: 6px; height: 6px; border-radius: 50%; background: #9fdbbd; box-shadow: 0 0 9px rgba(159, 219, 189, .56); }
.comment-source-card { display: grid; grid-template-columns: minmax(0, 1fr) auto; align-items: center; gap: 8px 14px; margin-top: 17px; border: 1px solid rgba(255, 255, 255, .1); border-radius: 11px; padding: 12px 14px; background: rgba(255, 255, 255, .025); }
.comment-source-copy { min-width: 0; }
.comment-source-title { color: rgba(246, 239, 255, .69); font-size: 10px; letter-spacing: .055em; }
.comment-source-title span { margin-left: 5px; color: rgba(233, 221, 248, .35); font-size: 9px; }
.comment-source-tags { display: flex; align-items: center; gap: 8px; margin-top: 7px; }
.comment-source-tag { display: inline-flex; align-items: center; gap: 6px; color: rgba(250, 246, 255, .78); font-size: 11px; }
.comment-source-tag > svg { width: 15px; height: 15px; fill: none; stroke: currentColor; stroke-width: 1.5; stroke-linecap: round; stroke-linejoin: round; }
.comment-source-tag > svg:first-child:not(:only-child) { color: #e5dcf3; }
.comment-source-tag .icon-cutout { fill: #211b31; stroke: none; }
.comment-source-divider { color: rgba(237, 224, 250, .3); font-size: 12px; }
.source-browser-mark { display: inline-grid; width: 15px; height: 15px; flex: 0 0 auto; place-items: center; border-radius: 50%; font: 700 10px/1 Arial, sans-serif; }
.source-browser-mark.browser-chrome { position: relative; border: 1px solid rgba(255, 255, 255, .4); background: conic-gradient(#e74b3b 0 34%, #45a857 34% 67%, #edc544 67%); }
.source-browser-mark.browser-chrome::after { width: 6px; height: 6px; border: 1px solid rgba(255, 255, 255, .75); border-radius: 50%; background: #4285f4; content: ''; }
.source-browser-mark.browser-edge { background: linear-gradient(145deg, #22c7a8, #2876d1 70%); color: white; font-style: italic; }
.source-browser-mark.browser-firefox { background: linear-gradient(145deg, #ffca54, #f07a32 48%, #a950dc); color: white; }
.source-browser-mark.browser-safari { background: radial-gradient(circle, #8bd7ff, #2787cf 72%); color: white; }
.source-browser-mark.browser-opera { background: #d94f6b; color: white; }
.source-browser-mark.browser-other { background: rgba(195, 179, 219, .4); color: #fff; }
.comment-source-button { display: inline-flex; align-items: center; justify-content: center; gap: 6px; border: 1px solid rgba(224, 207, 246, .18); border-radius: 100px; padding: 7px 10px; background: rgba(255, 255, 255, .035); color: rgba(244, 235, 255, .69); font: inherit; font-size: 9px; letter-spacing: .035em; white-space: nowrap; cursor: pointer; transition: border-color .2s, background .2s, color .2s; }
.comment-source-button:hover { border-color: rgba(224, 207, 246, .4); background: rgba(255, 255, 255, .08); color: #fff; }
.comment-source-button:focus-visible { outline: 2px solid rgba(230, 214, 255, .8); outline-offset: 2px; }
.comment-source-button svg { width: 13px; height: 13px; fill: none; stroke: currentColor; stroke-width: 1.5; stroke-linecap: round; stroke-linejoin: round; }
.comment-source-card > p { grid-column: 1 / -1; margin: 0; color: rgba(237, 228, 247, .39); font-size: 9px; line-height: 1.7; }
.comment-source-feedback { grid-column: 1 / -1; color: rgba(183, 226, 202, .82); font-size: 9px; }
.giscus { min-height: 240px; padding-top: 7px; }
.giscus-loading, .giscus-error { margin: 18px 0 0; color: rgba(237, 226, 248, .47); font-size: 11px; }
.giscus-error { color: #f1b9c1; }
.shared-comments-footnote { margin: 12px 0 0; padding-top: 13px; border-top: 1px solid rgba(255, 255, 255, .08); color: rgba(236, 227, 246, .39); font-size: 10px; line-height: 1.8; }
.comments-page-footer { padding: 19px 0 26px; border-top: 1px solid rgba(255, 255, 255, .1); color: rgba(242, 231, 250, .34); font-size: 9px; letter-spacing: .2em; }
.comments-page-footer span { margin: 0 7px; }
@media (max-width: 760px) {
  .comments-page-shell { width: calc(100% - 36px); }
  .comments-page-header { padding: 23px 0; }
  .comments-page-content { grid-template-columns: 1fr; gap: 32px; padding: 53px 0 55px; }
  .comments-intro { position: static; }
  .comments-intro h1 { margin: 17px 0 13px; font-size: clamp(27px, 8vw, 36px); line-height: 1.6; }
  .comments-intro p { max-width: 430px; font-size: 11px; }
  .shared-comments { padding: 19px 16px 15px; }
  .comment-source-card { grid-template-columns: minmax(0, 1fr); gap: 9px; padding: 11px; }
  .comment-source-button { justify-self: start; }
  .comment-source-card > p, .comment-source-feedback { grid-column: 1; }
  .comments-page-footer { font-size: 8px; letter-spacing: .12em; }
}
</style>
