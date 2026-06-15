<script setup lang="ts">
import { onMounted, onUnmounted } from "vue"
import ArtPlayer from "artplayer"
import Hls from "hls.js"

let player: ArtPlayer | null = null

onMounted(async () => {
  let url = ""
  try { url = (await (await fetch("/api/live/config")).json()).streamUrl } catch { /* ignore */ }
  if (!url) return
  await new Promise(r => setTimeout(r, 100)) // wait for DOM
  player = new ArtPlayer({
    container: "#artplayer-container",
    url,
    isLive: true,
    fullscreen: true,
    customType: {
      m3u8(video, url) {
        const hls = new Hls()
        hls.loadSource(url)
        hls.attachMedia(video)
      },
    },
  })
})

onUnmounted(() => player?.destroy())
</script>

<template>
  <div id="artplayer-container" class="live-page" />
</template>

<style>
.live-page { width: 100vw; height: 100vh; background: #000; overflow: hidden; }
.art-video-player { width: 100% !important; height: 100% !important; }
.art-video-player video { width: 100% !important; height: 100% !important; object-fit: contain !important; }
</style>
