<template>
  <div>
    <h1>绘制一个点2（P38）</h1>
    <GLCanvas ref="glCanvas" />
    <div>
      <i>通过传参的方式修改 x 坐标：</i>
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

  gl.clearColor(0.0, 0.0, 0.0, 1.0); // 以 RGBA 格式设置颜色，取值范围从 0.0 到 1.0

  // Initialize shaders
  const VSHADER_SOURCE =
    'attribute vec4 a_Position;\n' +
    'void main() {\n' +
    '  gl_Position = a_Position;\n' + // 设置顶点(vertex)坐标
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

  const a_Position = gl.getAttribLocation(gl.program, 'a_Position'); // 获取 attribute 变量地址
  if (a_Position < 0) {
    console.log("Failed to get the storage location of a_Position");
    return;
  }

  watchEffect(() => {
    gl.clear(gl.COLOR_BUFFER_BIT);
    gl.vertexAttrib3f(a_Position, cX.value, 0, 0); // 给 attribute 变量赋值
    gl.drawArrays(gl.POINTS, 0, 1); // 从第 1 个顶点开始绘制 1 个顶点
  })


})
</script>

<style lang="scss" scoped></style>