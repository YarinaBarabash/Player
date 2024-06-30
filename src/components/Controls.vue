<script setup lang="ts">

import Timeline from "@/components/Timeline.vue";

defineProps<{
  isPlaying: boolean
  selectedSpeed: string
  isFullScreen: boolean
  imageUrl: string
  isControlsShown: boolean
  duration: number
  currentTime: number
}>()

const emit = defineEmits<{
  (e: 'on-toggle-play-pause'): void
  (e: 'on-rewind-forward'): void
  (e: 'on-rewind-backward'): void
  (e: 'on-toggle-fullsize'): void
  (e: 'on-make-screenshot'): void
  (e: 'on-clear-screenshot'): void
  (e: 'on-update-current-time', newTime: number): void
  (e: 'on-update-selected-speed', speed: string): void
}>()

const togglePlayPause = () => {
  emit('on-toggle-play-pause')
}

const rewindForward = () => {
  emit('on-rewind-forward')
}

const rewindBackward = () => {
  emit('on-rewind-backward')
}

const updateSelectedSpeed = (event: Event) => {
  emit('on-update-selected-speed', (event.target as HTMLSelectElement).value)
}

const toggleFullSize = () => {
  emit('on-toggle-fullsize')
}

const makeScreenshot = () => {
  emit('on-make-screenshot')
}

const clearScreenshot = () => {
  emit('on-clear-screenshot')
}

const updateCurrentTime = (newTime: number) => {
  emit('on-update-current-time', newTime)
}
</script>

<template>
  <div class="controls">
    <button
        @click="togglePlayPause"
        class="play-button"
        aria-label="воспроизведение/пауза"
        title="воспроизведение/пауза"
    >
      <img v-show="!isPlaying" src="@/assets/img/play-button.svg" alt="Плей" >
      <img v-show="isPlaying" src="@/assets/img/pause-button.svg" alt="Пауза" >
    </button>
    <button
        class="rewind-forward-button"
        aria-label="перемотка вперед"
        title="перемотка вперед"
        @click="rewindForward"
    >
      <img src="@/assets/img/5-seconds-forward.svg" alt="+5">
    </button>
    <button
        class="rewind-backward-button"
        aria-label="перемотка назад"
        title="перемотка назад"
        @click="rewindBackward"
    >
      <img src="@/assets/img/5-seconds-back.svg" alt="-5">
    </button>
    <div class="selected">
      <span> Скорость воспроизведения</span>
      <select @change="updateSelectedSpeed" class="speed-select">
        <option value="1" >1</option>
        <option value="2">2</option>
        <option value="0.5">0.5</option>
      </select>
    </div>
    <button
        class="full-size-button"
        aria-label="Полноэкранный режим"
        title="Полноэкранный режим"
        @click="toggleFullSize"
    >
      <img src="@/assets/img/fullscreen.svg" alt="full-size" v-show="!isFullScreen">
      <img src="@/assets/img/fullscreen-exit.svg" alt="Exit full-size" v-show="isFullScreen">
    </button>
    <button
        class="make-screenshot-button"
        aria-label="Сделать снимок экрана"
        title="Сделать снимок экрана"
        @click="makeScreenshot"
    >
      <img src="@/assets/img/screenshot.svg" alt="Скриншот">
    </button>
    <a
        v-if="imageUrl"
        :href="imageUrl"
        download="скаченный скриншот"
        @click="clearScreenshot"
    >
      <button
          class="download-screenshot-button"
          aria-label="Скачать снимок экрана"
          title="Скачать снимок экрана"
      >
        <img src="@/assets/img/download-button.svg" alt="Скачать скриншот">
      </button>
    </a>
    <timeline
        class="timeline"
        v-if="isControlsShown"
        :duration="duration"
        :current-time="currentTime"
        @on-time-updated="updateCurrentTime"
    />
  </div>
</template>

<style scoped>
button {
  width: var(--button-width);
  height: var(--button-height);
  position: absolute;
  left: 0;
  top: 0;
  padding: 0;
  background-color: transparent;
  border: none;
}

img {
  width: var(--button-width);
  height: var(--button-height);
}

.full-size-button {
  top: 91%;
  left: 90%;
  width: max-content;
}

.rewind-forward-button{
  top: 50%;
  left: 85%;
}

.rewind-backward-button {
  top: 50%;
  left: 5%;
}

.selected{
  position: absolute;
  top: 90%;
  left: 5%;
  width: 110px;
}

.speed-select {
  width: 110px;
}

.play-button {
  top: 50%;
  left: 50%;
  border: none;
  display: flex;
  justify-content: center;
  align-items: center;
}

.make-screenshot-button {
  top: 91%;
  left: 80%;
  width: max-content;
}

.download-screenshot-button{
  top: 91%;
  left: 65%;
  width: max-content;
}

.timeline {
  position: absolute;
  bottom: 10px;
}
</style>