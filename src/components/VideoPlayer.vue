<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const videoRef = ref<HTMLVideoElement | null>(null)
const isPlaying = ref(false)
const currentTime = ref(0)
const duration = ref(0)
const volume = ref(1)
const showControls = ref(false)
let hideControlsTimeout: ReturnType<typeof setTimeout> | null = null

const formatTime = (time: number) => {
  const minutes = Math.floor(time / 60)
  const seconds = Math.floor(time % 60)
  return `${minutes}:${seconds.toString().padStart(2, '0')}`
}

const togglePlay = () => {
  if (!videoRef.value) return
  if (videoRef.value.paused) {
    videoRef.value.play()
    isPlaying.value = true
  } else {
    videoRef.value.pause()
    isPlaying.value = false
  }
}

const updateTime = () => {
  if (!videoRef.value) return
  currentTime.value = videoRef.value.currentTime
}

const changeTime = (event: Event) => {
  if (!videoRef.value) return
  const input = event.target as HTMLInputElement
  videoRef.value.currentTime = Number(input.value)
}

const changeVolume = (event: Event) => {
  if (!videoRef.value) return
  const input = event.target as HTMLInputElement
  videoRef.value.volume = Number(input.value)
  volume.value = videoRef.value.volume
}

const handleMouseEnter = () => {
  showControls.value = true
  if (hideControlsTimeout) clearTimeout(hideControlsTimeout)
}

const handleMouseLeave = () => {
  hideControlsTimeout = setTimeout(() => {
    showControls.value = false
  }, 1000)
}

onMounted(() => {
  if (!videoRef.value) return
  videoRef.value.onloadedmetadata = () => {
    duration.value = videoRef.value?.duration ?? 0
  }
})

onUnmounted(() => {
  if (hideControlsTimeout) clearTimeout(hideControlsTimeout)
})
</script>

<template>
  <div
    class="relative max-w-full max-h-screen overflow-hidden group rounded-lg cursor-pointer shadow-lg"
    @mouseenter="handleMouseEnter"
    @mouseleave="handleMouseLeave"
    @click="togglePlay"
  >
    <video
      ref="videoRef"
      src="../assets/1.mp4"
      @timeupdate="updateTime"
      class="w-full h-[500px] max-h-[100vh] object-contain"
    ></video>

    <!-- CONTROL PANEL -->
    <div
      class="absolute bottom-0 left-0 w-full bg-gray-900/40 p-4 flex items-center gap-4 transition-opacity duration-500 rounded-lg"
      :class="showControls ? 'opacity-100' : 'opacity-0'"
    >
      <button
        @click="togglePlay"
        class="text-white text-md p-1 rounded-lg bg-gray-700 hover:bg-gray-600 transition"
      >
        {{ isPlaying ? '⏸' : '▶' }}
      </button>

      <input
        type="range"
        :max="duration"
        :value="currentTime"
        @input="changeTime"
        class="absolute top-0 left-0 w-full h-1 bg-gray-600 appearance-none cursor-pointer"
      />
      <span class="text-white text-sm">
        {{ formatTime(currentTime) }} / {{ formatTime(duration) }}
      </span>

      <input
        type="range"
        min="0"
        max="1"
        step="0.1"
        :value="volume"
        @input="changeVolume"
        class="w-24 h-2 bg-gray-600 rounded-lg appearance-none cursor-pointer"
      />
    </div>
  </div>
</template>
