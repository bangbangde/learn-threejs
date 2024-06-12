<template>
  <canvas ref="canvas" width="300" height="300"></canvas>
</template>

<script setup>
import { ref, shallowRef, onMounted, defineExpose, defineProps } from "vue";

const props = defineProps(["width", "height"]);
const canvas = ref(null);
const gl = shallowRef(null);

defineExpose({
  canvas,
  gl
})

onMounted(() => {
  gl.value = canvas.value.getContext("webgl");
  canvas.value.width = props.width || 300;
  canvas.value.height = props.height || 300;
})
</script>

<style lang="scss" scoped></style>