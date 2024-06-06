<template>
  <div>
    <h1>清空绘图区（P16）</h1>
    <GLCanvas ref="glCanvas" />
    <div>
      <i>使用不同颜色清空画布：</i>
      <button v-for="(color, name) in colors" :key="name" @click="clear(color)">{{ name }}</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import GLCanvas from "./GLCanvas.vue";

const glCanvas = ref(null);
const colors = ref({
  red: [1, 0, 0, 1],
  green: [0, 1, 0, 1],
  blue: [0, 0, 1, 1],
})

function clear (color) {
  const gl = glCanvas.value.gl;
  gl.clearColor(...color);
  gl.clear(gl.COLOR_BUFFER_BIT);
}

onMounted(() => {
  const gl = glCanvas.value.gl;

  gl.clearColor(0.0, 0.0, 0.0, 1.0); // 以 RGBA 格式设置颜色，取值范围从 0.0 到 1.0
  gl.clear(gl.COLOR_BUFFER_BIT);     // 传入 gl.COLOR_BUFFER_BIT 常量，表示要清空的是“颜色缓冲区”
})
</script>

<style lang="scss" scoped></style>