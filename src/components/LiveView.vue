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
.live-page {
  position: fixed;
  inset: 0;
  background: #000;
  overflow: hidden;
}

/* Fill viewport — live-video-player has display: contents,
   so the skin renders as a direct child of .live-page */
live-video-skin {
  width: 100% !important;
  height: 100% !important;
}

/* hls-video is a custom element, NOT a native <video>,
   so the skin's internal ::slotted(video) selector doesn't match it.
   We must apply these styles explicitly. */
hls-video {
  width: 100% !important;
  height: 100% !important;
  object-fit: contain !important;
  display: block !important;
}

/* Reset border-radius on fullscreen player */
.media-default-skin--video {
  --media-border-radius: 0 !important;
}
</style>
