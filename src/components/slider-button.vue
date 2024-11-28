<template>
  <div class="button__slider">
      <div
       :style="sliderStyle"
       class="button__wrapper"
       :class="[props.buttonBackgroundColor]"
       ref="slider"
      >
       <span
          class="button__wrapper--text gradient-text center"
          :class="[props.titleStyle]"
          v-if="props.titleHTML"
          v-html="props.titleHTML"
       ></span>
       <span class="button__wrapper--text gradient-text" :class="[props.titleStyle]">
          {{ props.reverse ? props.titleReverse : props.title }}
       </span>
  
       <div
          :style="backgroundStyle"
          class="button__wrapper--background"
          :class="[props.backgroundColor]"
       ></div>
      </div>
  
      <button
       ref="button"
       @dragstart.prevent
       @click.stop.prevent="handleClick"
       @touchend.stop.prevent="endSlide"
       @touchstart.stop.prevent="startSlide"
       @touchmove.stop.prevent="moveSlide"
       :class="[props.active ? 'active' : '', props.sliderColor]"
       class="button__wrapper--slider"
       :style="buttonStyle"
      >
       <slot v-if="props.dopImage"></slot>
       <img
          v-else
          :style="props.reverse ? 'transform: scale(-1, -1)' : ''"
          class="button__wrapper--img"
          src="@/assets/swipe-arrow.svg"
          alt=""
       />
      </button>
  </div>
  
  </template>
  
  <script setup lang="ts">
  import { ref, defineProps, computed, defineEmits, watch } from 'vue';
  
  const props = defineProps<{
  dopImage?: boolean;
  buttonBackgroundColor?: string;
  titleStyle?: string;
  sliderColor?: string;
  backgroundColor?: string;
  title?: string;
  active?: boolean;
  refresh: boolean;
  reverse?: boolean;
  buttonWidth?: number;
  buttonHeight?: number;
  sliderHeight?: number;
  titleReverse?: string;
  toogle?: boolean;
  titleHTML?: string;
  }>();
  
  const emits = defineEmits(['setActive', 'update:refresh']);
  
  const slider = ref<HTMLDivElement | null>(null);
  const button = ref<HTMLButtonElement | null>(null);
  
  const sliderHeightDefault = 58;
  const buttonWidthDefault = 86;
  const buttonHeightDefault = 60;
  
  const getSliderWidth = computed(() => slider.value?.offsetWidth || 0);
  const getButtonWidth = computed(() => props.buttonWidth || buttonWidthDefault);
  const getButtonHeight = computed(() => props.buttonHeight || buttonHeightDefault);
  
  const touchDiff = ref(0);
  const endEvent = ref(false);
  const touchCordEnd = ref<number[]>([]);
  const touchCordStart = ref(0);
  let touchstartX = 0;
  
  const sliderStyle = computed(() => ({
  height: `${props.sliderHeight || sliderHeightDefault}px`,
  }));
  
  const backgroundStyle = computed(() => {
  const baseTransform = props.reverse
      ? `translateX(calc(${getSliderWidth.value}px))`
      : `translateX(calc(-100% + ${getButtonWidth.value}px))`;
  
  const touchTransform =
      touchDiff.value > -25 && touchstartX
       ? `translateX(calc(${props.reverse ? touchDiff.value + getButtonWidth.value : `-100% + ${getButtonWidth.value}px + ${touchDiff.value}`}px))`
       : '';
  
  const endTransform = endEvent.value
      ? `translateX(calc(-100% +${getSliderWidth.value}px))`
      : '';
  
  return {
      transform: touchTransform || endTransform || baseTransform,
  };
  });
  
  const buttonStyle = computed(() => {
  const baseTransform = props.reverse
      ? `translate(calc(${getSliderWidth.value - getButtonWidth.value}px), -50%)`
      : `translate(calc(-100% + ${getButtonWidth.value}px), -50%)`;
  
  const touchTransform =
      touchDiff.value > -25 && touchstartX
       ? `translate(${touchDiff.value}px, -50%)`
       : '';
  
  const endTransform = endEvent.value
      ? props.reverse
       ? `translate(calc(-100% + ${getButtonWidth.value}px), -50%)`
       : `translate(${getSliderWidth.value - getButtonWidth.value}px, -50%)`
      : '';
  
  return {
      transform: touchTransform || endTransform || baseTransform,
      width: `${getButtonWidth.value}px`,
      height: `${getButtonHeight.value}px`,
  };
  });
  
  const startSlide = (event: TouchEvent) => {
  touchCordStart.value = event.changedTouches[0].screenX;
  touchstartX = event.touches[0].clientX;
  };
  
  const handleClick = async () => {
  if (/iPhone|iPad|iPod|Android/i.test(navigator.userAgent)) {
      return;
  } else {

       endEvent.value = true;
       emits('setActive');
      
  }
  };
  
  const endSlide = async (event: TouchEvent) => {

      if (props.reverse) {
       endSlideLeft(event);
      } else {
       endSlideRight(event);
      }
  
  };
  
  const endSlideRight = (event: TouchEvent) => {
  if (touchCordEnd.value[1] < touchCordEnd.value[0]) {
      if (getSliderWidth.value - touchDiff.value < 150) {
       touchDiff.value = getSliderWidth.value - getButtonWidth.value;
       emits('setActive');
      } else {
       touchDiff.value = 0;
      }
  } else {
      endEvent.value = false;
      touchDiff.value = 0;
  }
  touchCordEnd.value = [];
  };
  
  const endSlideLeft = (event: TouchEvent) => {
  if (touchCordEnd.value[1] > touchCordEnd.value[0]) {
      if (touchDiff.value < 40) {
       endEvent.value = true;
       emits('setActive');
      }
  } else {
      endEvent.value = false;
  }
  touchCordEnd.value = [];
  touchDiff.value = 0;
  touchstartX = 0;
  };
  
  const moveSlide = (event: TouchEvent) => {
  if (props.reverse) {
      moveSlideLeft(event);
  } else {
      moveSlideRight(event);
  }
  };
  
  const moveSlideRight = (event: TouchEvent) => {
  if (props.active) {
      const touchY = event.touches[0].clientX;
      if (touchDiff.value >= 0 && touchDiff.value <= getSliderWidth.value - getButtonWidth.value) {
       touchDiff.value = touchY - touchstartX;
       touchCordEnd.value.unshift(touchDiff.value);
      }
  }
  };
  
  const moveSlideLeft = (event: TouchEvent) => {
  if (props.active) {
      const touchY = event.touches[0].clientX;
      if (touchDiff.value > -22) {
       touchDiff.value =
          touchY - (touchstartX - getSliderWidth.value + getButtonWidth.value) < -22
           ? -22
           : touchY - (touchstartX - getSliderWidth.value + getButtonWidth.value);
       touchCordEnd.value.unshift(touchDiff.value);
      }
  }
  };
  
  const refresh = computed({
  get() {
      return props.refresh;
  },
  set(newValue) {
      emits('update:refresh', newValue);
  },
  });
  
  watch(refresh, () => {
  if (refresh.value) {
      endEvent.value = false;
      touchCordEnd.value = [];
      touchDiff.value = 0;
      refresh.value = false;
      touchstartX = 0;
  }
  });
  </script>
  
  <style lang="scss">
  .gradient-text {
  font-size: 24px;
  font-weight: bold;
  background: linear-gradient(90deg, #000000, #555555, #aaaaaa, #ffffff, #aaaaaa, #555555, #000000);
  background-size: 200% auto;
  background-clip: text;
  -webkit-background-clip: text;
  color: transparent !important;
  animation: gradient-animation 3s linear infinite;
  }
  @keyframes gradient-animation {
  0% {
      background-position: 200% 50%;
  }
  100% {
      background-position: 0% 50%;
  }
  }
  .button__slider {
  position: relative;
  height: 60px;
  width: 100%;
  display: flex;
  align-items: center;
  }
  .button__wrapper {
  position: relative;
  border-radius: 1.5625rem;
  overflow: hidden;
  background-color: #e6e6e6;
  height: 58px;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  
  &--background {
      position: absolute;
      top: 0;
      width: calc(100% + 86px);
      height: 100%;
      background: linear-gradient(93.38deg, #cd10ff -21.6%, #8a01a0 55.07%, #69dbff 106.83%);
  }
  
  &--text {
      overflow: hidden;
      font-size: 1.5rem;
      position: relative;
      font-weight: 700;
      line-height: 1.1;
      text-align: center;
      color: #343330;
      @media (max-width: 429px) {
       font-size: 1rem;
      }
      &.center {
       text-align: center;
      }
  }
  
  &--slider {
      display: flex;
      align-items: center;
      justify-content: center;
      border-radius: 1.5625rem;
      border: 0.0625rem solid white;
      position: absolute;
      top: 50%;
      height: 60px;
      width: 86px;
      background-color: #e6e6e6;
      &.active {
       background-color: #cd10ff;
      }
  }
  }
  </style>
