<script setup lang="ts">
import { ref, onMounted, onUnmounted, nextTick } from "vue"
import videojs from "video.js"
import "video.js/dist/video-js.css"

const API = "/api"
const streamUrl = ref("")
const loading = ref(true)
const error = ref("")
const videoRef = ref<HTMLVideoElement | null>(null)
let player: ReturnType<typeof videojs> | null = null

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
  player?.dispose()
})

function initPlayer() {
  if (!streamUrl.value || !videoRef.value) return
  player = videojs(videoRef.value, {
    autoplay: true,
    muted: true,
    controls: true,
    fluid: true,
    liveui: true,
    responsive: true,
    sources: [{ src: streamUrl.value, type: "application/x-mpegURL" }],
  })
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
    <div v-else class="video-js-wrap">
      <video ref="videoRef" class="video-js vjs-theme-city" playsinline />
    </div>
  </div>
</template>

<style scoped>
.live-page {
  width: 100vw;
  height: 100vh;
  background: #000;
  overflow: hidden;
}
.live-status {
  text-align: center;
  color: rgba(255, 255, 255, 0.6);
}
.video-js-wrap {
  width: 100%;
  height: 100%;
}
</style>

<style>
.live-page .video-js {
  width: 100%;
  height: 100%;
}
/* City skin — dark glass-like UI */
.vjs-theme-city {
  --vjs-theme-city--primary: #fff;
  --vjs-theme-city--secondary: rgba(255,255,255,0.15);
}
.vjs-theme-city .vjs-control-bar {
  background: linear-gradient(transparent, rgba(0,0,0,0.7));
  padding-top: 24px;
}
.vjs-theme-city .vjs-play-progress {
  background: linear-gradient(90deg, #4fc3f7, #ab47bc);
}
.vjs-theme-city .vjs-volume-level {
  background: linear-gradient(90deg, #4fc3f7, #ab47bc);
}
</style>
