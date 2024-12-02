<template>
  <div class="video-container">
    <video ref="videoRef" class="video-player" controls></video>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, watch } from 'vue'
import Hls from 'hls.js'

const props = defineProps({
  url: {
    type: String,
    required: true
  },
  poster: {
    type: String,
    default: ''
  }
})

const emit = defineEmits(['play', 'pause', 'error'])
const videoRef = ref(null)
let hls = null

const initPlayer = () => {
  if (!videoRef.value) return

  // 销毁现有的 HLS 实例
  if (hls) {
    hls.destroy()
  }

  // 检查浏览器是否支持 HLS
  if (Hls.isSupported()) {
    hls = new Hls({
      debug: false,
      enableWorker: true
    })
    
    hls.attachMedia(videoRef.value)
    hls.on(Hls.Events.MEDIA_ATTACHED, () => {
      hls.loadSource(props.url)
    })

    // 错误处理
    hls.on(Hls.Events.ERROR, (event, data) => {
      if (data.fatal) {
        switch (data.type) {
          case Hls.ErrorTypes.NETWORK_ERROR:
            console.error('网络错误，尝试恢复...')
            hls.startLoad()
            break
          case Hls.ErrorTypes.MEDIA_ERROR:
            console.error('媒体错误，尝试恢复...')
            hls.recoverMediaError()
            break
          default:
            console.error('无法恢复的错误')
            hls.destroy()
            break
        }
        emit('error', data)
      }
    })
  } else if (videoRef.value.canPlayType('application/vnd.apple.mpegurl')) {
    // 对于原生支持 HLS 的浏览器（如 Safari）
    videoRef.value.src = props.url
  }
}

// 监听 URL 变化
watch(() => props.url, () => {
  initPlayer()
})

// 事件监听
onMounted(() => {
  const video = videoRef.value
  if (video) {
    video.addEventListener('play', () => emit('play'))
    video.addEventListener('pause', () => emit('pause'))
    initPlayer()
  }
})

// 清理
onBeforeUnmount(() => {
  if (hls) {
    hls.destroy()
  }
})
</script>

<style scoped>
.video-container {
  width: 100%;
  aspect-ratio: 16/9;
  background: #000;
}

.video-player {
  width: 100%;
  height: 100%;
}
</style>