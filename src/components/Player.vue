<script setup lang="ts">
import {onBeforeUnmount, onMounted, ref, watch} from "vue";
import Timeline from "@/components/Timeline.vue";

const videoEl = ref<HTMLVideoElement>();
const playerEl = ref<HTMLDivElement>();
const selectedSpeed = ref<string>('1');
const currentTime = ref<number>(0);
const imageURL = ref<string>('');
const isControlsShown = ref<boolean>(false);
const isPlaying= ref<boolean>(false);
const isFullScreen = ref<boolean>(false);


let timeoutId: number;

const REWIND_TIME = 5
const TIMEOUT_HIDDEN_CONTROLS = 2000

const togglePlayPause = () => {
  if (!isPlaying.value) {
    videoEl.value.play()
    isPlaying.value = true
  } else {
    videoEl.value.pause()
    isPlaying.value = false
  }
}

const rewindForward = () => {
  videoEl.value.currentTime += REWIND_TIME
  currentTime.value += REWIND_TIME
}


const rewindBackward = () => {
  videoEl.value.currentTime -= REWIND_TIME
  currentTime.value -= REWIND_TIME
}

const changeSpeed = (newValue: string) => {
  videoEl.value.playbackRate = Number(newValue)
}

const requestFullScreen = async () => {
  if (playerEl.value.requestFullscreen) {
    await playerEl.value.requestFullscreen();
  } else if (videoEl.value.webkitRequestFullscreen) {
    await videoEl.value.webkitRequestFullscreen();
  } else if (videoEl.value.webkitEnterFullScreen) {
    await videoEl.value.webkitEnterFullScreen();
  }
}

const toggleFullSize = () => {
  playerEl.value.focus()

  if (!document.fullscreenElement) {
    requestFullScreen()
    isFullScreen.value = true
  } else {
    document.exitFullscreen()
    isFullScreen.value = false
  }
}

const makeScreenshot = () => {
  const canvas = document.createElement("canvas");
  const ctx = canvas.getContext("2d");

  ctx.drawImage( videoEl.value, 0, 0, canvas.width, canvas.height );
  imageURL.value = canvas.toDataURL('image/jpeg');
}
const clearScreenshot = () => {
  imageURL.value = ''
}

const hideControlsWithTimeout = () => {
  if (timeoutId) {
    clearTimeout(timeoutId)
  }

  timeoutId = setTimeout(() => {
    isControlsShown.value = false
  }, TIMEOUT_HIDDEN_CONTROLS)
}

const showControls = () => {
  isControlsShown.value = true;
  hideControlsWithTimeout()
}

const onSpaceClick = () => {
  togglePlayPause();
  showControls()
}

const onTimeUpdate = (event: Event) => {
  currentTime.value = (event.target as HTMLVideoElement).currentTime
}

const onFullscreenChangeHandler = () => {
  if(!document.fullscreenElement) {
    isFullScreen.value = false
  }
}

const updateCurrentTime = (newCurrentTime: number) => {
  videoEl.value.currentTime = newCurrentTime;
  currentTime.value = newCurrentTime;
}

onMounted(()=> {
  playerEl.value.focus()
  document.addEventListener('fullscreenchange', onFullscreenChangeHandler);
  currentTime.value = videoEl.value.currentTime;
});

onBeforeUnmount(() => {
  document.removeEventListener('fullscreenchange', onFullscreenChangeHandler);
})

watch(selectedSpeed, changeSpeed)
</script>

<template>
  <div
    class="player"
    ref="playerEl"
    tabindex="0"
    @mouseenter="showControls"
    @mousemove="showControls"
    @keydown.space="onSpaceClick"
    @click="showControls"
    @keydown.right="rewindForward"
    @keydown.left="rewindBackward"
    @keydown.f="toggleFullSize"
  >

    <video
        ref="videoEl"
        crossorigin="anonymous"
        poster="@/assets/img/statham.webp"
        :class="{ fullScreen: isFullScreen }"
        playsinline
        allowfullscreen
        @timeupdate="onTimeUpdate"
        @click="togglePlayPause"
    >
      <source
          src="http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4"
          type="video/mp4"
      >
    </video>

    <transition>
      <div class="controls" v-show="isControlsShown">
        <button
            @click="togglePlayPause"
            class="play-button"
            aria-label="воспроизведение/пауза"
            title="воспроизведение/пауза"
        >
          <img v-show="!isPlaying" src="@/assets/img/play-button.svg" alt="Плей" >
          <img v-show="isPlaying" src="@/assets/img/pause-button.svg" alt="Пауза" >
        </button>
        <button class="rewind-forward-button"
                aria-label="перемотка вперед"
                title="перемотка вперед"
                @click="rewindForward"
        >
          <img src="@/assets/img/5-seconds-forward.svg" alt="+5">
        </button>
        <button class="rewind-backward-button"
                aria-label="перемотка назад"
                title="перемотка назад"
                @click="rewindBackward"
        >
          <img src="@/assets/img/5-seconds-back.svg" alt="-5">
        </button>
        <div class="selected">
          <span> Скорость воспроизведения</span>
          <select v-model="selectedSpeed" class="speed-select">
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
        <a v-if="imageURL"
           :href="imageURL"
           download="скаченный скриншот"
           @click="clearScreenshot"
        >
          <button class="download-screenshot-button"
                  aria-label="Скачать снимок экрана"
                  title="Скачать снимок экрана"
          >
            <img src="@/assets/img/download-button.svg" alt="Скачать скриншот">
          </button>
        </a>
        <timeline class="timeline"
                  v-if="isControlsShown"
                  :duration="videoEl?.duration"
                  :current-time="currentTime"
                  @on-time-updated="updateCurrentTime"/>
      </div>
    </transition>
  </div>
</template>

<style scoped>

.player {
  position: relative;
  width: max-content;
  outline: none;
}

video {
  border-radius: 15px;
  object-fit: cover;
  width: 1024px;
  height: 720px;
}

button {
  width: 45px;
  height: 45px;
  position: absolute;
  left: 0;
  top: 0;
  padding: 0;
  background-color: transparent;
  border: none;
}

img {
  width: var(--icon-width);
  height: var(--icon-height);
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

video::-webkit-media-controls {
  display: none !important;
}
.fullScreen {
  width: 100%;
  height: 100%;
}
</style>`