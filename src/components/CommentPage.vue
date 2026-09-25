<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref } from 'vue'

const homeHref = `${import.meta.env.BASE_URL}`
const commentMount = ref<HTMLElement | null>(null)
const loadState = ref<'loading' | 'ready' | 'error'>('loading')
let observer: MutationObserver | undefined

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
  .comments-page-footer { font-size: 8px; letter-spacing: .12em; }
}
</style>
