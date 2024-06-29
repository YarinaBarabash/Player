<script lang="ts" setup>
import { defineProps, computed, ref } from 'vue'

const timelineEl = ref<HTMLDivElement>()

const clickOnTimeline = (event: MouseEvent) => {
  const currentTimeInPercent = event.clientX / timelineEl.value.clientWidth;
  const currentTime = currentTimeInPercent * props.duration

  emit('on-time-updated', currentTime);
}
const props = defineProps<{
  currentTime: number,
  duration: number
}>();

const emit = defineEmits<{
  (e: 'on-time-updated', newCurrentTime: number): void;
}>()
const currentTimeInPercent = computed(() => {
  if (props.currentTime === 0) {
    return 0;
  }

 return `${props.currentTime * 100 / props.duration}%`
})
</script>

<template>
<div class="timeline" @click="clickOnTimeline" ref="timelineEl">
  <div :style="{ width: currentTimeInPercent }" class="timeline-current"></div>
</div>
</template>

<style scoped>

.timeline {
  position: relative;
  width: 100%;
  height: 6px;
  background: var(--timeline);
}
.timeline-current {
  background: var(--active-timeline);
  height: 6px;
}

</style>