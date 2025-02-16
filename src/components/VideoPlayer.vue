<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const videoRef = ref<HTMLVideoElement | null>(null)
const isPlaying = ref(false)
const currentTime = ref(0)
const duration = ref(0)
const volume = ref(1)
const showControls = ref(false)
const tooltipVisible = ref(false)
const tooltipX = ref(0)
let hideControlsTimeout: ReturnType<typeof setTimeout> | null = null

const toggleFullscreen = () => {
  if (!videoRef.value) return
  if (!document.fullscreenElement) {
    videoRef.value.requestFullscreen()
  } else {
    document.exitFullscreen()
  }
}

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

const showTooltip = (event: MouseEvent) => {
  tooltipVisible.value = true
  const slider = event.currentTarget as HTMLInputElement
  tooltipX.value = event.offsetX - slider.offsetWidth / 2 + 12
}

const hideTooltip = () => {
  tooltipVisible.value = false
}
</script>

<template>
  <div
    class="relative max-w-full max-h-screen overflow-hidden rounded-lg shadow-lg group"
    @mouseenter="handleMouseEnter"
    @mouseleave="handleMouseLeave"
  >
    <video
      ref="videoRef"
      src="../assets/1.mp4"
      @timeupdate="updateTime"
      @click="togglePlay"
      class="w-full h-[500px] max-h-[100vh] object-contain"
    ></video>

    <!-- CONTROL PANEL -->
    <div
      class="absolute bottom-0 left-0 flex items-center w-full gap-4 p-4 transition-opacity duration-500 rounded-lg bg-gray-900/10"
      :class="showControls ? 'opacity-100' : 'opacity-0'"
    >
      <button
        @click="togglePlay"
        class="p-1 transition rounded-lg cursor-pointer text-sky-50 bg-sky-50 text-md hover:bg-sky-100"
      >
        <img class="w-4" :src="isPlaying ? 'pause.svg' : 'play.svg'" alt="play" />
      </button>

      <input
        type="range"
        :max="duration"
        :value="currentTime"
        @input="changeTime"
        class="absolute top-0 left-0 w-full h-1 appearance-none cursor-pointer bg-sky-50 [&::-webkit-slider-runnable-track]:w-full [&::-webkit-slider-runnable-track]:h-1 [&::-webkit-slider-runnable-track]:bg-sky-50 [&::-webkit-slider-runnable-track]:rounded-full [&::-webkit-slider-thumb]:appearance-none [&::-webkit-slider-thumb]:w-3 [&::-webkit-slider-thumb]:h-3 [&::-webkit-slider-thumb]:bg-[#222222] [&::-webkit-slider-thumb]:rounded-full [&::-webkit-slider-thumb]:translate-y-[-3px] [&::-moz-range-track]:w-full [&::-moz-range-track]:h-1 [&::-moz-range-track]:bg-sky-50 [&::-moz-range-track]:rounded-full [&::-moz-range-thumb]:w-3 [&::-moz-range-thumb]:h-3 [&::-moz-range-thumb]:bg-[#222222] [&::-moz-range-thumb]:rounded-full [&::-moz-range-thumb]:translate-y-[-1px]"
      />
      <span class="text-xs font-semibold text-stone-50">
        {{ formatTime(currentTime) }} / {{ formatTime(duration) }}
      </span>

      <div class="relative flex items-center">
        <input
          type="range"
          min="0"
          max="1"
          step="0.01"
          :value="volume"
          @input="changeVolume"
          class="w-20 appearance-none cursor-pointer [&::-webkit-slider-runnable-track]:w-full [&::-webkit-slider-runnable-track]:h-1 [&::-webkit-slider-runnable-track]:bg-sky-50 [&::-webkit-slider-runnable-track]:rounded-full [&::-webkit-slider-thumb]:appearance-none [&::-webkit-slider-thumb]:w-3 [&::-webkit-slider-thumb]:h-3 [&::-webkit-slider-thumb]:bg-[#222222] [&::-webkit-slider-thumb]:rounded-full [&::-webkit-slider-thumb]:translate-y-[-3px] [&::-moz-range-track]:w-full [&::-moz-range-track]:h-1 [&::-moz-range-track]:bg-sky-50 [&::-moz-range-track]:rounded-full [&::-moz-range-thumb]:w-3 [&::-moz-range-thumb]:h-3 [&::-moz-range-thumb]:bg-[#222222] [&::-moz-range-thumb]:rounded-full [&::-moz-range-thumb]:translate-y-[-1px]"
          @mousemove="showTooltip"
          @mouseleave="hideTooltip"
          ref="slider"
        />

        <!-- Tooltip -->
        <div
          v-if="tooltipVisible"
          :style="{ left: tooltipX + 'px' }"
          class="absolute top-[-30px] px-2 py-1 text-xs text-stone-50 bg-black rounded-xl"
        >
          {{ `Vol ` + Math.floor(volume * 100) }}
        </div>
      </div>

      <!-- Fullscreen -->
      <button @click="toggleFullscreen" class="p-1 transition rounded-lg cursor-pointer text-md">
        <img class="w-5" src="/screen.svg" alt="fullscreen" />
      </button>
    </div>
  </div>
</template>
