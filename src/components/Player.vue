<script setup lang="ts">

import {onBeforeUnmount, onMounted, ref, watch} from "vue";
import Timeline from "@/components/Timeline.vue";

const isPlaying= ref(false);
const video = ref<HTMLVideoElement>();
const selected = ref<string>('1');
const imageURL = ref<string | null>(null);
const isShowing = ref(false);
const playerEl = ref<HTMLDivElement>();
const isFullScreen = ref(false);
const currentTime = ref(0);

let timeoutId: number;


const isPlayPause = () => {
  if (!isPlaying.value) {
    video.value.play()
    isPlaying.value = true
  } else {
    video.value.pause()
    isPlaying.value = false
  }
}

const rewindForward = () => {
  video.value.currentTime += 5
  currentTime.value += 5
}


const rewindBackward = () => {
  video.value.currentTime -= 5
  currentTime.value -= 5
}

const speed = (newValue: string) => {
  video.value.playbackRate = Number(newValue)
}

watch(selected, speed)

const requestFullScreen = async () => {
  if (playerEl.value.requestFullscreen) {
    await playerEl.value.requestFullscreen();
  } else if (video.value.webkitRequestFullscreen) {
    await video.value.webkitRequestFullscreen();
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
  ctx.drawImage( video.value, 0, 0, canvas.width, canvas.height );

  imageURL.value = canvas.toDataURL('image/jpeg');

}
const clearScreenshot = () => {
  imageURL.value = ''
}

const hiddenControls = () => {
  if (timeoutId) {
    clearTimeout(timeoutId)
  }
  timeoutId = setTimeout(() => {
    isShowing.value = false
  }, 2000)

}

const onMouseover = () => {
  isShowing.value = true;
  hiddenControls()
}
const onClickSpace = () => {
  isPlayPause();
  onMouseover()
}

const onTimeupdate = (event: Event) => {
  currentTime.value = (event.target as HTMLVideoElement).currentTime
}

const fullscreenChangeHandler = () => {
  if(!document.fullscreenElement) {
    isFullScreen.value = false
  }
}

const updateCurrentTime = (newCurrentTime: number) => {
  video.value.currentTime = newCurrentTime;
  currentTime.value = newCurrentTime;
  console.log("currentTime", newCurrentTime);
}

onMounted(()=> {
  playerEl.value.focus()
  document.addEventListener('fullscreenchange', fullscreenChangeHandler);
  currentTime.value = video.value.currentTime;
});

onBeforeUnmount(() => {
  document.removeEventListener('fullscreenchange', fullscreenChangeHandler);
})

</script>

<template>
  <div class="player"
       ref="playerEl"
       @keydown.space="onClickSpace"
       tabindex="0"
       @mouseenter="onMouseover"
       @mousemove="onMouseover"
       @click="onMouseover"
       @keydown.right="rewindForward"
       @keydown.left="rewindBackward"
       @keydown.f="toggleFullSize"
  >
    <video
        ref="video"
        width="1024"
        height="720"
        crossorigin="anonymous"
        poster="@/assets/img/statham.webp"
        @click="isPlayPause"
        playsinline
        webkit-playsinline
        :class="{ fullScreen: isFullScreen }"
        @timeupdate="onTimeupdate"
    >
      <source
          src="http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4"
          type="video/mp4"
      >
    </video>
    <Transition>
      <div class="controls" v-show="isShowing">
        <button @click="isPlayPause" class="play-button">
          <img src="@/assets/img/play-button.svg" alt="Плей" v-show="!isPlaying">
          <img src="@/assets/img/pause-button.svg" alt="Пауза" v-show="isPlaying">
        </button>
        <button @click="rewindForward" class="rewindForward">
          <img src="@/assets/img/5-seconds-forward.svg" alt="+5">
        </button>
        <button @click="rewindBackward" class="rewindBackward">
          <img src="@/assets/img/5-seconds-back.svg" alt="-5">
        </button>
        <div class="selected">
          <span> Скорость воспроизведения</span>
          <select v-model="selected" class="speed">
            <option value="1" >1</option>
            <option value="2">2</option>
            <option value="0.5">0.5</option>
          </select>
        </div>
        <button @click="toggleFullSize" class="fullSizeButton">
          <img src="@/assets/img/fullscreen.svg" alt="full-size" v-show="!isFullScreen">
          <img src="@/assets/img/fullscreen-exit.svg" alt="Exit full-size" v-show="isFullScreen">
        </button>
        <button @click="makeScreenshot" class="makeScreenshotButton">
          <img src="@/assets/img/screenshot.svg" alt="Скриншот">
        </button>
        <a v-if="imageURL" :href="imageURL" download="скаченный скриншот" @click="clearScreenshot">
          <button class="downloadScreenshotButton">
            <img src="@/assets/img/download-button.svg" alt="Скачать скриншот">
          </button>
        </a>
        <timeline class="timeline" :duration="video?.duration" :current-time="currentTime" @on-time-updated="updateCurrentTime"/>
      </div>
    </Transition>
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
  width: 45px;
  height: 45px;
}

.fullSizeButton {
  top: 91%;
  left: 90%;
  width: max-content;
}

.rewindForward{
  top: 50%;
  left: 85%;
}
.rewindBackward {
  top: 50%;
  left: 5%;
}

.selected{
  position: absolute;
  top: 90%;
  left: 5%;
  width: 110px;
}

.speed {
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

.makeScreenshotButton{
  top: 91%;
  left: 80%;
  width: max-content;
}

.downloadScreenshotButton{
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