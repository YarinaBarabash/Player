<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref, watch } from "vue";
import Controls from "@/components/Controls.vue";

const REWIND_TIME = 5
const TIMEOUT_HIDDEN_CONTROLS = 2000

const videoEl = ref<HTMLVideoElement>();
const playerEl = ref<HTMLDivElement>();

const selectedSpeed = ref<string>('1');
const currentTime = ref<number>(0);
const imageURL = ref<string>('');
const isControlsShown = ref<boolean>(false);
const isPlaying= ref<boolean>(false);
const isFullScreen = ref<boolean>(false);

let timeoutId: number;

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
      @timeupdate="onTimeUpdate"
      @click="togglePlayPause"
    >
      <source
          src="http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4"
          type="video/mp4"
      >
    </video>

    <transition>
      <controls
          v-if="isControlsShown"
          :duration="videoEl?.duration || 0"
          :current-time="currentTime"
          :is-playing="isPlaying"
          :image-url="imageURL"
          :selected-speed="selectedSpeed"
          :is-full-screen="isFullScreen"
          :is-controls-shown="isControlsShown"
          @on-toggle-play-pause="togglePlayPause"
          @on-clear-screenshot="clearScreenshot"
          @on-make-screenshot="makeScreenshot"
          @on-rewind-backward="rewindBackward"
          @on-rewind-forward="rewindForward"
          @on-toggle-fullsize="toggleFullSize"
          @on-update-current-time="updateCurrentTime"
          @on-update-selected-speed="changeSpeed"
      />
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

video::-webkit-media-controls {
  display: none !important;
}
.fullScreen {
  width: 100%;
  height: 100%;
}
</style>`