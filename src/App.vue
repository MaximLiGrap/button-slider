<template>
<sliderButton v-model:refresh="refresh"></sliderButton>
</template>

<script lang="ts" setup>
import { ref } from 'vue';
import sliderButton from './components/slider-button.vue';
const refresh = ref(false)
</script>