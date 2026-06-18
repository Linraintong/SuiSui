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
/* live-video-player 默认 display:contents 不产生盒子，
   改为 block + 尺寸使其撑满容器，为内部 skin 提供布局上下文 */
live-video-player {
  display: block;
  width: 100%;
  height: 100%;
}

live-video-skin {
  width: 100% !important;
  height: 100% !important;
}

/* hls-video 是自定义元素，库内 ::slotted(video) 匹配不到它，
   所以需要显式约束尺寸让视频正常留存在流中，不撑大容器。
   pointer-events:none 让触摸事件穿透到 media-container，
   使库的手势系统(pointerdown/pointerup)能正常检测点击显示控件 */
.live-page hls-video {
  display: block !important;
  width: 100% !important;
  height: 100% !important;
  object-fit: contain !important;
  pointer-events: none !important;
}
</style>
