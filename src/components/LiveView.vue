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
  <div class="live-page">
    <live-video-player>
      <live-video-skin>
        <hls-video :src="streamUrl" playsinline />
      </live-video-skin>
    </live-video-player>
  </div>
</template>

<style>
.live-page {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  overflow: hidden;
  background: #000;
  --media-border-radius: 0;
}
live-video-skin {
  width: 100% !important;
  height: 100% !important;
}
/* 覆盖 hls-video 默认的 display:contents，
   改为绝对定位使其脱离布局流、填满视口 */
.live-page hls-video {
  display: block !important;
  position: absolute !important;
  top: 0 !important;
  left: 0 !important;
  width: 100% !important;
  height: 100% !important;
}
</style>
