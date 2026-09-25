<script setup lang="ts">
import { computed, nextTick, onBeforeUnmount, ref } from 'vue'

const publicAsset = (path: string) => `${import.meta.env.BASE_URL}${path}`

const backgrounds = [
  // 后期更换歌曲时，只需要修改对应背景下的 track.src / track.name。
  { name: 'bg1', image: publicAsset('backgrounds/bg1.png'), track: { name: '左转灯', src: publicAsset('music/left-turn-lamp.mp3') } },
  { name: 'haha1', image: publicAsset('backgrounds/haha1.png'), track: { name: 'winter luv', src: publicAsset('music/winter-luv.mp3') } },
  { name: 'jk', image: publicAsset('backgrounds/jk.png'), track: { name: 'エミュレーション', src: publicAsset('music/emulation.mp3') } },
  { name: 'kosaki', image: publicAsset('backgrounds/kosakihd.png'), track: { name: 'Forget it', src: publicAsset('music/forget-it.mp3') } },
]

const quoteSets = [
  {
    lines: ['昨日見た夢の続きはどこへ', 'きっとまた行けるから準備している', '永遠の閉じ込めた声を残して', '瞬間を犠牲にしてシャッターうを押す'],
    signature: 'Cherry blossom Fall We Drift Away',
  },
  {
    lines: ['The night keeps every secret', 'until the light finds it.'],
    signature: 'I was here.',
  },
  {
    lines: ['If the stars remember us', 'we were never lost.'],
    signature: 'Let the silence bloom.',
  },
  {
    lines: ['私の名前は小野寺小咲', '中学三年生の私は、恋をしていた','相手は同じクラスの一条楽君','勉強もスポーツも取り立てて目立ったことはない彼だけど','そんな中にある素朴な優しさが、私はとても好きだった'],
    signature: '我的名字是...',
  },
]

const currentBackground = ref(0)
const isSwitching = ref(false)
const audio = ref<HTMLAudioElement | null>(null)
const isPlaying = ref(false)
const progress = ref(0)
const duration = ref(0)
let transitionTimer: ReturnType<typeof setTimeout> | undefined

const activeBackground = computed(() => backgrounds[currentBackground.value])
const activeQuote = computed(() => quoteSets[currentBackground.value])
const audioSource = computed(() => activeBackground.value.track.src)
const progressPercent = computed(() => duration.value ? (progress.value / duration.value) * 100 : 0)

async function selectBackground(index: number) {
  if (index === currentBackground.value) return
  const wasPlaying = isPlaying.value
  currentBackground.value = index
  isSwitching.value = true
  if (transitionTimer) clearTimeout(transitionTimer)
  transitionTimer = setTimeout(() => {
    isSwitching.value = false
  }, 850)
  progress.value = 0
  duration.value = 0
  isPlaying.value = false
  await nextTick()
  audio.value?.load()
  if (wasPlaying && audio.value) {
    await audio.value.play()
    isPlaying.value = true
  }
}

function chooseBackground(index: number) {
  void selectBackground(index)
}

async function toggleAudio() {
  if (!audio.value) return
  if (audio.value.paused) {
    await audio.value.play()
    isPlaying.value = true
  } else {
    audio.value.pause()
    isPlaying.value = false
  }
}

function updateProgress() {
  if (!audio.value) return
  progress.value = audio.value.currentTime
  duration.value = audio.value.duration || 0
}

function seek(event: Event) {
  if (!audio.value || !duration.value) return
  const value = Number((event.target as HTMLInputElement).value)
  audio.value.currentTime = value
  progress.value = value
}

function finishAudio() {
  isPlaying.value = false
  progress.value = 0
}

onBeforeUnmount(() => {
  if (transitionTimer) clearTimeout(transitionTimer)
})
</script>

<template>
  <div class="single-screen" :class="{ switching: isSwitching }">
    <Transition name="background-fade" mode="out-in">
      <div
        :key="activeBackground.image"
        class="background-layer"
        :style="{ backgroundImage: `url(${activeBackground.image})` }"
        aria-hidden="true"
      ></div>
    </Transition>
    <div class="background-vignette" aria-hidden="true"></div>
    <div class="background-grain" aria-hidden="true"></div>

    <section class="quote-overlay" aria-label="页面引言">
      <p v-for="line in activeQuote.lines" :key="line">{{ line }}</p>
      <p class="quote-signature">{{ activeQuote.signature }}</p>
    </section>

    <div class="background-navigation" aria-label="背景切换">
      <div class="background-dots">
        <button
          v-for="(background, index) in backgrounds"
          :key="background.image"
          class="background-dot"
          :class="{ active: currentBackground === index }"
          :aria-label="`切换到${background.name}`"
          :title="background.name"
          @click="chooseBackground(index)"
        ></button>
      </div>
    </div>

    <aside class="music-player" aria-label="音乐播放器">
      <audio
        ref="audio"
        :src="audioSource"
        @loadedmetadata="updateProgress"
        @timeupdate="updateProgress"
        @ended="finishAudio"
      ></audio>
      <input
        class="progress-range"
        :class="{ visible: isPlaying }"
        type="range"
        min="0"
        :max="duration || 1"
        step="0.01"
        :value="progress"
        :style="{ '--progress': `${progressPercent}%` }"
        aria-label="播放进度"
        @input="seek"
      />
      <div class="player-topline">
        <button class="music-control play-control" :aria-label="isPlaying ? '暂停' : '播放'" :title="activeBackground.track.name" @click="toggleAudio">
          <svg v-if="!isPlaying" class="play-icon" viewBox="0 0 24 24" aria-hidden="true"><path d="M8.5 5.3 18.2 12l-9.7 6.7V5.3Z" /></svg>
          <svg v-else class="pause-icon" viewBox="0 0 24 24" aria-hidden="true"><rect x="7" y="6" width="3.3" height="12" rx="1.2" /><rect x="13.7" y="6" width="3.3" height="12" rx="1.2" /></svg>
        </button>
        <span class="track-name">{{ activeBackground.track.name }}</span>
      </div>
    </aside>
  </div>
</template>
