<template>
  <div style="height: 100%; overflow: auto;">
    <h1>使用鼠标绘制点 - uniform</h1>
    <GLCanvas ref="glCanvas" @mousedown="handleMousedown" />
    <div>
      <button @click="clear">clear</button>
      <ol>
        <li v-for="(item, i) in points" :key="i">{{ item[0] }}</li>
      </ol>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import GLCanvas from "./GLCanvas.vue";
import { initShaders } from "@/lib/webgl-helpers.js";

const glCanvas = ref(null);
const points = ref([]);

function clear() {
  const gl = glCanvas.value.gl;
  gl.clear(gl.COLOR_BUFFER_BIT);
  points.value = [];
}

function handleMousedown(ev) {
  const {offsetWidth, offsetHeight} = ev.target;
  const x = (ev.offsetX - offsetWidth / 2) / (offsetWidth / 2);
  const y = (-ev.offsetY + offsetHeight / 2) / (offsetHeight / 2);
  console.log({x, y});
  const gl = glCanvas.value.gl;
  const point = [[x, y, 0],[ev.offsetX / offsetWidth, ev.offsetY / offsetHeight,0,1]];
  points.value.push(point);
  gl.clear(gl.COLOR_BUFFER_BIT);
  points.value.forEach((point) => {
    gl.$setAPosition(...point[0]);
    gl.$setUFragColor(...point[1]);
    gl.drawArrays(gl.POINTS, 0, 1); // 必须在这里同步地 draw 才能将多个点都绘制出来
  })
  // gl.drawArrays(gl.POINTS, 0, points.value.length); // 猜测错误，在这里 draw 只会绘制一个点
}

function initGL() {
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
    'precision mediump float;\n' +
    'uniform vec4 u_FragColor;\n' +  // uniform 变量
    'void main() {\n' +
    '  gl_FragColor = u_FragColor;\n' +
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

  const u_FragColor = gl.getUniformLocation(gl.program, 'u_FragColor'); // 获取 uniform 变量地址
  // 不同于 getAttribLocation 失败返回 -1，这里失败会返回 null
  if (!u_FragColor) {
    console.log("Failed to get the storage location of u_FragColor");
    return;
  }

  gl.$setAPosition = (...value) => {
    gl.vertexAttrib3f(a_Position, ...value);
  }

  gl.$setUFragColor = (...value) => {
    gl.uniform4f(u_FragColor, ...value);
  }

  gl.clear(gl.COLOR_BUFFER_BIT);
}

onMounted(() => {
  initGL();
})
</script>

<style lang="scss" scoped></style>