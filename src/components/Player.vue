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

const isMobile = () => {
  let check = false;
  (function(a){if(/(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge |maemo|midp|mmp|mobile.+firefox|netfront|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows ce|xda|xiino/i.test(a)||/1207|6310|6590|3gso|4thp|50[1-6]i|770s|802s|a wa|abac|ac(er|oo|s\-)|ai(ko|rn)|al(av|ca|co)|amoi|an(ex|ny|yw)|aptu|ar(ch|go)|as(te|us)|attw|au(di|\-m|r |s )|avan|be(ck|ll|nq)|bi(lb|rd)|bl(ac|az)|br(e|v)w|bumb|bw\-(n|u)|c55\/|capi|ccwa|cdm\-|cell|chtm|cldc|cmd\-|co(mp|nd)|craw|da(it|ll|ng)|dbte|dc\-s|devi|dica|dmob|do(c|p)o|ds(12|\-d)|el(49|ai)|em(l2|ul)|er(ic|k0)|esl8|ez([4-7]0|os|wa|ze)|fetc|fly(\-|_)|g1 u|g560|gene|gf\-5|g\-mo|go(\.w|od)|gr(ad|un)|haie|hcit|hd\-(m|p|t)|hei\-|hi(pt|ta)|hp( i|ip)|hs\-c|ht(c(\-| |_|a|g|p|s|t)|tp)|hu(aw|tc)|i\-(20|go|ma)|i230|iac( |\-|\/)|ibro|idea|ig01|ikom|im1k|inno|ipaq|iris|ja(t|v)a|jbro|jemu|jigs|kddi|keji|kgt( |\/)|klon|kpt |kwc\-|kyo(c|k)|le(no|xi)|lg( g|\/(k|l|u)|50|54|\-[a-w])|libw|lynx|m1\-w|m3ga|m50\/|ma(te|ui|xo)|mc(01|21|ca)|m\-cr|me(rc|ri)|mi(o8|oa|ts)|mmef|mo(01|02|bi|de|do|t(\-| |o|v)|zz)|mt(50|p1|v )|mwbp|mywa|n10[0-2]|n20[2-3]|n30(0|2)|n50(0|2|5)|n7(0(0|1)|10)|ne((c|m)\-|on|tf|wf|wg|wt)|nok(6|i)|nzph|o2im|op(ti|wv)|oran|owg1|p800|pan(a|d|t)|pdxg|pg(13|\-([1-8]|c))|phil|pire|pl(ay|uc)|pn\-2|po(ck|rt|se)|prox|psio|pt\-g|qa\-a|qc(07|12|21|32|60|\-[2-7]|i\-)|qtek|r380|r600|raks|rim9|ro(ve|zo)|s55\/|sa(ge|ma|mm|ms|ny|va)|sc(01|h\-|oo|p\-)|sdk\/|se(c(\-|0|1)|47|mc|nd|ri)|sgh\-|shar|sie(\-|m)|sk\-0|sl(45|id)|sm(al|ar|b3|it|t5)|so(ft|ny)|sp(01|h\-|v\-|v )|sy(01|mb)|t2(18|50)|t6(00|10|18)|ta(gt|lk)|tcl\-|tdg\-|tel(i|m)|tim\-|t\-mo|to(pl|sh)|ts(70|m\-|m3|m5)|tx\-9|up(\.b|g1|si)|utst|v400|v750|veri|vi(rg|te)|vk(40|5[0-3]|\-v)|vm40|voda|vulc|vx(52|53|60|61|70|80|81|83|85|98)|w3c(\-| )|webc|whit|wi(g |nc|nw)|wmlb|wonu|x700|yas\-|your|zeto|zte\-/i.test(a.substr(0,4))) check = true;})(navigator.userAgent||navigator.vendor||window.opera);
  return check;
};

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
  // if (playerEl.value.requestFullscreen && !isMobile()) {
  //   await playerEl.value.requestFullscreen();
  // } else if (video.value.webkitRequestFullscreen) {
    await video.value.webkitRequestFullscreen();
  // }
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
        :class="{ fullScreen: isFullScreen && !isMobile() }"
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