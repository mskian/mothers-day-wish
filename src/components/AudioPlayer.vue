<template>
    <div class="audio-player mb-5">
      <audio ref="audioRef" :src="src" preload="auto" autoplay loop></audio>
  
      <p class="audio-title">{{ title }}</p>
  
      <button class="play-button" @click="togglePlay" aria-label="Play/Pause">
        <component :is="isPlaying ? Pause : Play" :size="32" />
      </button>
  
      <div class="progress-container">
        <div class="progress-bar" :style="{ width: progress + '%' }"></div>
      </div>
  
      <div class="duration-container">
        <span>{{ formatTime(currentTime) }}</span>
        <span>{{ formatTime(duration) }}</span>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted, onUnmounted } from 'vue'
  import { Play, Pause } from 'lucide-vue-next'
  
  defineProps({
    src: String,
    title: String
  })
  
  const audioRef = ref(null)
  const isPlaying = ref(false)
  const progress = ref(0)
  const duration = ref(0)
  const currentTime = ref(0)
  
  const formatTime = (time) => {
    const mins = Math.floor(time / 60)
    const secs = String(Math.floor(time % 60)).padStart(2, '0')
    return `${mins}:${secs}`
  }
  
  const updateProgress = () => {
    const audio = audioRef.value
    if (!audio?.duration) return
    currentTime.value = audio.currentTime
    progress.value = (audio.currentTime / audio.duration) * 100
  }
  
  const updatePlayStatus = () => {
    const audio = audioRef.value
    isPlaying.value = audio && !audio.paused
  }
  
  const togglePlay = () => {
    const audio = audioRef.value
    if (!audio) return
    audio.paused ? audio.play() : audio.pause()
  }
  
  onMounted(() => {
    const audio = audioRef.value
    if (!audio) return
  
    audio.addEventListener('loadedmetadata', () => {
      duration.value = audio.duration || 0
    })
    audio.addEventListener('timeupdate', updateProgress)
    audio.addEventListener('play', updatePlayStatus)
    audio.addEventListener('pause', updatePlayStatus)
  })
  
  onUnmounted(() => {
    const audio = audioRef.value
    if (!audio) return
  
    audio.removeEventListener('timeupdate', updateProgress)
    audio.removeEventListener('play', updatePlayStatus)
    audio.removeEventListener('pause', updatePlayStatus)
  })
  </script>
  
<style scoped>
.audio-player {
  max-width: 400px;
  margin: auto;
  padding: 1.5rem;
  border-radius: 1.5rem;
  background: linear-gradient(to right, #1e3c72, #2a5298);
  color: #fff;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
  text-align: center;
  font-family: "Anek Tamil", sans-serif;
  transition: all 0.3s ease;
}
.audio-title {
  font-size: 1.2rem;
  font-weight: 600;
  margin-bottom: 1.2rem;
  word-break: break-word;
}
.play-button {
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto;
  width: 64px;
  height: 64px;
  background: #fff;
  color: #1e3c72;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  transition: transform 0.2s ease, background 0.2s ease;
}
.play-button:hover {
  transform: scale(1.1);
  background: #f0f0f0;
}
.play-button svg {
  width: 32px;
  height: 32px;
}
.progress-container {
  background: rgba(255, 255, 255, 0.2);
  height: 10px;
  border-radius: 5px;
  margin: 1.5rem 0 1rem;
  overflow: hidden;
}
.progress-bar {
  background: linear-gradient(to right, #00c6ff, #0072ff);
  height: 100%;
  transition: width 0.2s ease;
  border-radius: 5px;
}
.duration-container {
  display: flex;
  justify-content: space-between;
  font-size: 0.9rem;
  opacity: 0.85;
  margin-top: 0.5rem;
}
</style>