<script setup lang="ts">
import { onMounted, onUnmounted, ref, watch, nextTick } from "vue"
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

function fitVideo() {
  const video = videoRef.value
  if (!video) return
  video.style.width = "100vw"
  video.style.height = "100vh"
}

watch(streamUrl, (url) => {
  const video = videoRef.value
  if (!video || !url) return
  video.removeAttribute("src")
  if (Hls.isSupported()) {
    const hls = new Hls()
    hls.on(Hls.Events.MANIFEST_PARSED, fitVideo)
    hls.loadSource(url)
    hls.attachMedia(video)
  } else if (video.canPlayType("application/vnd.apple.mpegurl")) {
    video.src = url
    video.oncanplay = fitVideo
  }
})

// Force video to viewport size, re-apply on any resize
let ro: ResizeObserver | null = null
onMounted(() => {
  nextTick(fitVideo)
  ro = new ResizeObserver(fitVideo)
  ro.observe(document.documentElement)
})
onUnmounted(() => ro?.disconnect())
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
.live-page,
live-video-skin {
  width: 100% !important;
  height: 100% !important;
}
.live-page video {
  width: 100% !important;
  height: 100% !important;
  object-fit: contain !important;
}
</style>
