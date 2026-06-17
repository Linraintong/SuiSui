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
  // Don't set src on the video element — HLS.js handles it or native
  if (Hls.isSupported()) {
    video.removeAttribute("src")
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
  inset: 0;
  background: #000;
  overflow: hidden;
  --media-border-radius: 0;
}
live-video-player,
live-video-skin {
  width: 100vw !important;
  height: 100vh !important;
}
.live-page video {
  display: block !important;
  width: 100vw !important;
  height: 100vh !important;
  object-fit: contain !important;
}
</style>
