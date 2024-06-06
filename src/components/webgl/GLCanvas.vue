<template>
  <canvas ref="canvas"></canvas>
</template>

<script setup>
import { ref, shallowRef, onMounted, defineExpose } from "vue";

const canvas = ref(null);
const gl = shallowRef(null);

defineExpose({
  canvas,
  gl
})

onMounted(() => {
  gl.value = canvas.value.getContext("webgl");
})
</script>

<style lang="scss" scoped></style>