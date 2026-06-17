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
        <video ref="videoRef" :src="streamUrl" playsinline />
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
}

live-video-player {
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
