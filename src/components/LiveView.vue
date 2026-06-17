<script setup lang="ts">
import { onMounted, ref } from "vue"
import "@videojs/html/live-video"
import "@videojs/html/media/hls-video"

const streamUrl = ref("")

onMounted(async () => {
  try {
    const res = await fetch("/api/live/config")
    if (res.ok) {
      const config = await res.json()
      streamUrl.value = config.streamUrl ?? ""
    }
  } catch { /* ignore */ }
})
</script>

<template>
  <live-video-player class="live-page">
    <live-video-skin>
      <hls-video :src="streamUrl" playsinline />
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

hls-video {
  display: block !important;
  width: 100% !important;
  height: 100% !important;
  object-fit: contain !important;
}

.media-default-skin--video {
  --media-border-radius: 0 !important;
}
</style>
