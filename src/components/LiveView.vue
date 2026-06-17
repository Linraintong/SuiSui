<script setup lang="ts">
import { onMounted, ref } from "vue"
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

// Use hls.js when the stream URL changes
import { watch } from "vue"

watch(streamUrl, (url) => {
  const video = videoRef.value
  if (!video || !url) return
  if (Hls.isSupported()) {
    const hls = new Hls()
    hls.loadSource(url)
    hls.attachMedia(video)
  } else if (video.canPlayType("application/vnd.apple.mpegurl")) {
    // Native HLS (Safari)
    video.src = url
  }
})
</script>

<template>
  <live-video-player class="live-page">
    <live-video-skin>
      <video ref="videoRef" :src="streamUrl" playsinline />
    </live-video-skin>
  </live-video-player>
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
  width: 100vw;
  height: 100vh;
  height: -webkit-fill-available;
  background: #000;
  overflow: hidden;
  z-index: 9999;
}

/* Override display:contents so the player creates its own box */
live-video-player {
  display: block !important;
  width: 100% !important;
  height: 100% !important;
}

live-video-skin {
  width: 100% !important;
  height: 100% !important;
}

.media-default-skin--video {
  --media-border-radius: 0 !important;
}
</style>
