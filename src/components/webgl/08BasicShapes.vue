<template>
  <div style="height: 100%; overflow: auto;">
    <h1>webgl可以直接绘制的基本图形</h1>
    <GLCanvas ref="glCanvas" @mousedown="handleMousedown" />
    <div class="form">
      <select v-model="targetShape">
        <option v-for="option in options" :key="option.value" :value="option.value">
          {{ option.text }}
        </option>
      </select>
      <button @click="clear">clear</button>
      <br>
      <img src="@/assets/imgs/basic-shapes.png" width="500" alt="webgl basic shapes">
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from "vue";
import GLCanvas from "./GLCanvas.vue";
import { initShaders } from "@/lib/webgl-helpers.js";

const glCanvas = ref(null);
const targetShape = ref(null);
const options = ref([]);
const points = ref([]);

function handleMousedown(ev) {
  const {offsetWidth, offsetHeight} = ev.target;
  const x = (ev.offsetX - offsetWidth / 2) / (offsetWidth / 2);
  const y = (-ev.offsetY + offsetHeight / 2) / (offsetHeight / 2);
  const point = [x, y];
  points.value.push(point);
}

function clear() {
  points.value.splice(0, points.value.length);
}

function draw() {
  if (!glCanvas.value?.gl) return;

  const gl = glCanvas.value.gl;
  gl.$clear();
  if (!points.value?.length) return;
  const n = gl.$initVertexBuffers(points.value);
  gl.drawArrays(gl.POINTS, 0, n);
  gl.drawArrays(targetShape.value, 0, n);
}

watch([targetShape, points], draw, { deep: true });

onMounted(() => {
  const gl = glCanvas.value.gl;

  options.value = [
    { text: "点(gl.POINTS)", value: gl.POINTS },
    { text: "线段(gl.LINES)", value: gl.LINES },
    { text: "线条(gl.LINE_STRIP)", value: gl.LINE_STRIP },
    { text: "回路(gl.LINE_LOOP)", value: gl.LINE_LOOP },
    { text: "三角形(gl.TRIANGLES)", value: gl.TRIANGLES },
    { text: "三角带(gl.TRIANGLE_STRIP)", value: gl.TRIANGLE_STRIP },
    { text: "三角扇(gl.TRIANGLE_FAN)", value: gl.TRIANGLE_FAN },
  ];

  targetShape.value = options.value[0].value;

  // 顶点着色器
  const VSHADER_SOURCE = 
    'attribute vec4 a_Position;\n' +
    'void main() {\n' +
    '  gl_Position = a_Position;\n' +
    '  gl_PointSize = 10.0;\n' +
    '}\n';

  // 片元着色器
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

  gl.$clear = () => {
    gl.clearColor(0.0, 0.0, 0.0, 1.0);
    gl.clear(gl.COLOR_BUFFER_BIT);
  }

  gl.$initVertexBuffers = function (points) {
    const vertices = new Float32Array(points.flat());
    const n = points.length;
    const vertexBuffer = gl.createBuffer();
    if (!vertexBuffer) {
      console.log("Failed to create the buffer object");
      return -1;
    }
    gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
    gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);
    gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 0, 0);
    gl.enableVertexAttribArray(a_Position);
    return n;
  }
  gl.$clear();
});
</script>

<style lang="scss" scoped></style>