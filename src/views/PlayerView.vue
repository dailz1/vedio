<template>
    <div class="player-page">
      <h1>{{ currentVideo.title }}</h1>
      
      <VideoPlayer
        :url="currentVideo.url"
        :poster="currentVideo.poster"
        :options="playerOptions"
        @play="onPlay"
        @pause="onPause"
        @error="onError"
      />
      
      
    </div>
  </template>
  
  <script setup>
  import { ref, watchEffect } from 'vue'
  import VideoPlayer from '../components/VideoPlayer.vue'
  
  // 播放列表数据
  const playlist = ref([
    {
      id: 1,
      title: '年会直播',
      url: 'http://192.168.1.112:8080/hls/test119.m3u8',
      poster: '/poster1.jpg'
    },
    
  ])
  
  const currentVideo = ref(playlist.value[0])
  
  // 播放器配置
  const playerOptions = {
    theme: '#ff6b6b',  // 自定义主题色
    autoplay: false,   // 禁用自动播放
    settings: [
      {
        name: '画质',
        key: 'quality',
        value: '720p',
        type: 'select',
        options: [
          { value: '1080p', text: '1080P' },
          { value: '720p', text: '720P' },
          { value: '480p', text: '480P' }
        ]
      }
    ]
  }
  
  // 事件处理
  const onPlay = () => {
    console.log('视频开始播放')
  }
  
  const onPause = () => {
    console.log('视频暂停')
  }
  
  const onError = (error) => {
    console.error('播放错误：', error)
  }
  
  const changeVideo = (video) => {
    currentVideo.value = video
  }

  // 添加标题更新逻辑
watchEffect(() => {
  document.title = `${currentVideo.value.title}` // 可以自定义标题格式
})

  </script>
  
  <style scoped>
  .player-page {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
  }
  
  .playlist {
    margin-top: 20px;
    border: 1px solid #eee;
    padding: 10px;
  }
  </style>