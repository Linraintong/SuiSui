<script setup lang="ts">
import { onMounted, ref, watch } from "vue"
import "@videojs/html/live-video"
import Hls from "hls.js"

const streamUrl = ref("")
const videoRef = ref<HTMLVideoElement | null>(null)

onMounted(async () => {
  try {
    const res = await fetch("/api/live/config")
    if (res.ok) {
      const config = await res.json()
      streamUrl.value = config.streamUrl ?? ""
    }
  } catch { /* ignore */ }
})

watch(streamUrl, (url) => {
  const video = videoRef.value
  if (!video || !url) return
  video.removeAttribute("src")
  if (Hls.isSupported()) {
    const hls = new Hls()
    hls.loadSource(url)
    hls.attachMedia(video)
  } else if (video.canPlayType("application/vnd.apple.mpegurl")) {
    video.src = url
  }
})
</script>

<template>
  <div class="live-page">
    <live-video-player>
      <live-video-skin>
        <video ref="videoRef" playsinline />
      </live-video-skin>
    </live-video-player>
  </div>
</template>

<style>
html, body {
  margin: 0 !important;
  padding: 0 !important;
  height: 100% !important;
  overflow: hidden !important;
}
.live-page {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: #000;
  --media-border-radius: 0;
}
live-video-skin {
  width: 100% !important;
  height: 100% !important;
}

/* 绝对定位视频，完全脱离布局流，
   其原生分辨率再也不会撑大容器 */
.live-page video {
  position: absolute !important;
  top: 0 !important;
  left: 0 !important;
  width: 100% !important;
  height: 100% !important;
  object-fit: contain !important;
}
</style>
