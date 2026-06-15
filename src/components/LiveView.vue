<script setup lang="ts">
import { ref, onMounted } from "vue"

// Register Vidstack web components
import "vidstack/elements"
// Core player styles
import "vidstack/player/styles/base.css"

const API = "/api"
const streamUrl = ref("")
const loading = ref(true)
const error = ref("")

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
})
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
    <div v-else-if="error" class="live-status">
      <p class="text-body-2 text-medium-emphasis">{{ error }}</p>
    </div>
    <media-player v-else class="player-container" :src="streamUrl" :live="true" stream-type="live" view-type="video" :controls="true">
      <media-provider>
        <video></video>
      </media-provider>
    </media-player>
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
.player-container {
  display: block;
  width: 100%;
  height: 100%;
}
</style>
