<template>
  <div>
    <h1>绘制一个点（P22）</h1>
    <GLCanvas ref="glCanvas" />
    <div>
      <i>修改 x 坐标：</i>
      <input type="range" max="1" min="-1" step="0.1" v-model="cX">
      <i>{{ cX }}</i>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watchEffect } from "vue";
import GLCanvas from "./GLCanvas.vue";
import { initShaders } from "@/lib/webgl-helpers.js";

const glCanvas = ref(null);
const cX = ref(0);



onMounted(() => {
  const gl = glCanvas.value.gl;

  watchEffect(async () => {
    gl.clearColor(0.0, 0.0, 0.0, 1.0); // 以 RGBA 格式设置颜色，取值范围从 0.0 到 1.0
    gl.clear(gl.COLOR_BUFFER_BIT);     // 传入 gl.COLOR_BUFFER_BIT 常量，表示要清空的是“颜色缓冲区”

    // Initialize shaders
    const VSHADER_SOURCE =
      'void main() {\n' +
      `  gl_Position = vec4(${cX.value}, 0.0, 0.0, 1.0);\n` + // 设置顶点(vertex)坐标
      '  gl_PointSize = 10.0;\n' +
      '}\n';

    const FSHADER_SOURCE =
      'void main() {\n' +
      '  gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);\n' +
      '}\n';

    if (!initShaders(gl, VSHADER_SOURCE, FSHADER_SOURCE)) {
      console.log('Failed to intialize shaders.');
      return;
    }


    gl.drawArrays(gl.POINTS, 0, 1); // 从第 1 个顶点开始绘制 1 个顶点
  })
})
</script>

<style lang="scss" scoped></style>