<template>
  <div style="height: 100%; overflow: auto;">
    <h1>使用鼠标绘制点- 探索多次调用 drawArrays 绘制多点的表现</h1>
    <p>在 demo04 中，在一个 for 循环中多次调用 drawArrays，可以把这些顶点信息全部设置 webgl 缓冲区中，从而绘制出所有点。</p>
    <p>本例使用 setTimeout 依次调用表现为只会绘制出最后设置的一个点。</p>
    <p><i>目前猜测是不是 js 事件循环一轮会实际触发 webgl 的 drawArrays。</i></p>
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
  points.value.forEach((point, i) => {
    setTimeout(() => {
      gl.$setAPosition(...point[0]);
      gl.$setUFragColor(...point[1]);
      gl.drawArrays(gl.POINTS, 0, 1);
    }, i * 1000);
  })
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