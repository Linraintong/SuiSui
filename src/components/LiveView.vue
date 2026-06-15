<script setup lang="ts">
import { ref, onMounted, onUnmounted, nextTick } from "vue"
import Hls from "hls.js"

const API = "/api"
const streamUrl = ref("")
const loading = ref(true)
const error = ref("")
const videoRef = ref<HTMLVideoElement | null>(null)
let hls: Hls | null = null

onMounted(async () => {
  try {
    const r = await fetch(`${API}/live/config`)
    if (r.ok) {
      const data = await r.json()
      streamUrl.value = data.streamUrl || ""
    }
  } catch {
    error.value = "无法加载直播配置"
  }
  loading.value = false
  await nextTick()
  initPlayer()
})

onUnmounted(() => {
  hls?.destroy()
})

function initPlayer() {
  if (!streamUrl.value || !videoRef.value) return
  const vid = videoRef.value
  if (Hls.isSupported()) {
    hls = new Hls()
    hls.loadSource(streamUrl.value)
    hls.attachMedia(vid)
    hls.on(Hls.Events.MANIFEST_PARSED, () => {
      vid.play().catch(() => {})
    })
  } else if (vid.canPlayType("application/vnd.apple.mpegurl")) {
    vid.src = streamUrl.value
  }
}
</script>

<template>
  <div class="live-page">
    <div v-if="loading" class="live-status">
      <p class="text-body-2 text-medium-emphasis mt-2">加载直播...</p>
    </div>
    <div v-else-if="!streamUrl" class="live-status">
      <v-icon size="48" color="rgba(255,255,255,0.3)" class="mb-2">mdi-video-off-outline</v-icon>
      <p class="text-body-2 text-medium-emphasis mt-3">直播流未配置</p>
      <p class="text-caption text-medium-emphasis mt-1">请管理员在后台配置直播流地址</p>
    </div>
    <div v-else-if="error" class="live-status">
      <p class="text-body-2 text-medium-emphasis">{{ error }}</p>
    </div>
    <video v-else ref="videoRef" class="player-video" controls autoplay playsinline muted />
  </div>
</template>

<style scoped>
.live-page {
  width: 100vw;
  height: 100vh;
  background: #000;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}
.live-status {
  text-align: center;
  color: rgba(255, 255, 255, 0.6);
}
.player-video {
  width: 100%;
  height: 100%;
  object-fit: contain;
  display: block;
}
</style>
