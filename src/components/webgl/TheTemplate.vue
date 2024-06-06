<template>
  <div>
    <h1>DEMO名称</h1>
    <GLCanvas ref="glCanvas" />
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import GLCanvas from "./GLCanvas.vue";

const glCanvas = ref(null);

onMounted(() => {
  const gl = glCanvas.value.gl;

  gl.clearColor(0.0, 0.0, 0.0, 1.0); // 以 RGBA 格式设置颜色，取值范围从 0.0 到 1.0
  gl.clear(gl.COLOR_BUFFER_BIT);     // 传入 gl.COLOR_BUFFER_BIT 常量，表示要清空的是“颜色缓冲区”
})
</script>

<style lang="scss" scoped></style>