<script setup lang="ts">
import { onMounted, ref } from "vue"
import "@videojs/html/live-video"
import "@videojs/html/media/hls-video"

const streamUrl = ref("")
const isTouch = ref(false)

onMounted(async () => {
  // 检测触摸设备
  isTouch.value = "ontouchstart" in window || navigator.maxTouchPoints > 0

  // 加载直播流地址
  try {
    const res = await fetch("/api/live/config")
    if (res.ok) {
      const config = await res.json()
      streamUrl.value = config.streamUrl ?? ""
    }
  } catch { /* ignore */ }

  if (!isTouch.value) return

  // ── 触摸设备：绕过被 <video> 拦截的指针事件 ──────────────────────
  // 手势系统在 media-container 上监听 pointerdown/pointerup，
  // 但 hls-video（继承 HTMLVideoElement）在手机上会拦截这些事件。
  // 这里直接通过 Shadow DOM 强制显示控件，并用手势 touch 事件替代。
  const waitForPlayer = setInterval(() => {
    const skin = document.querySelector("live-video-skin")
    const root = skin?.shadowRoot
    if (!root) return
    clearInterval(waitForPlayer)

    const container = root.querySelector("media-container")
    if (!container) return

    // 始终显示控件（用户可直接点击按钮）
    container.setAttribute("data-visible", "")

    // 点击画面时切换控件显隐
    const livePage = document.querySelector(".live-page")
    livePage?.addEventListener("touchend", () => {
      container.toggleAttribute("data-visible")
    }, { passive: true })
  }, 200)
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
