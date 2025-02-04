<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const videoRef = ref<HTMLVideoElement | null>(null)
const isPlaying = ref(false)
const currentTime = ref(0)
const duration = ref(0)
const volume = ref(1)
const showControls = ref(true)
let hideControlsTimeout: ReturnType<typeof setTimeout> | null = null

const togglePlay = () => {
  if (!videoRef.value) return
  if (videoRef.value.paused) {
    videoRef.value.play()
    isPlaying.value = true
  } else {
    videoRef.value.pause()
    isPlaying.value = false
  }
  resetControlsTimeout()
}

const updateTime = () => {
  if (!videoRef.value) return
  currentTime.value = videoRef.value.currentTime
}

const changeTime = (event: Event) => {
  if (!videoRef.value) return
  const input = event.target as HTMLInputElement
  videoRef.value.currentTime = Number(input.value)
  resetControlsTimeout()
}

const changeVolume = (event: Event) => {
  if (!videoRef.value) return
  const input = event.target as HTMLInputElement
  videoRef.value.volume = Number(input.value)
  volume.value = videoRef.value.volume
  resetControlsTimeout()
}

const resetControlsTimeout = () => {
  showControls.value = true
  if (hideControlsTimeout) clearTimeout(hideControlsTimeout)
  hideControlsTimeout = setTimeout(() => {
    showControls.value = false
  }, 1000)
}

const handleMouseMove = () => {
  resetControlsTimeout()
}

onMounted(() => {
  if (!videoRef.value) return
  duration.value = videoRef.value.duration
  resetControlsTimeout()
  window.addEventListener('mousemove', handleMouseMove)
})

onUnmounted(() => {
  if (hideControlsTimeout) clearTimeout(hideControlsTimeout)
  window.removeEventListener('mousemove', handleMouseMove)
})
</script>

<template>
  <div class="relative max-w-full max-h-screen overflow-hidden group rounded-lg cursor-pointer">
    <video
      ref="videoRef"
      src="../assets/1.mp4"
      @timeupdate="updateTime"
      class="w-full h-[500px] max-h-[100vh] object-contain"
    ></video>

    <!-- OVLÁDACÍ PANEL -->
    <div
      class="absolute bottom-0 left-0 w-full bg-gray-900/80 p-4 flex items-center gap-4 transition-opacity duration-500 rounded-lg"
      :class="showControls ? 'opacity-100' : 'opacity-0'"
    >
      <button
        @click="togglePlay"
        class="text-white text-xl p-2 rounded-lg bg-gray-700 hover:bg-gray-600 transition"
      >
        {{ isPlaying ? '⏸' : '▶' }}
      </button>

      <input
        type="range"
        :max="duration"
        :value="currentTime"
        @input="changeTime"
        class="w-full h-2 bg-gray-600 rounded-lg appearance-none cursor-pointer"
      />

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
