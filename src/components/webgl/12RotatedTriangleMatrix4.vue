<template>
  <div>
    <h1>画一个三角形 - 使用矩阵lib</h1>
    <GLCanvas ref="glCanvas" />
    <input type="range" v-model="angle" min="0" max="360" step="1">
    <div style="display: flex; flex-direction: column; flex-wrap: wrap;line-height: 40px;width: 160px;height: 160px;">
      <span v-for="(item, i) in matrix" :key="i" style="width: 40px; height: 40px; text-align: center; overflow: hidden;">{{ item }}</span>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from "vue";
import GLCanvas from "./GLCanvas.vue";
import { initShaders } from "@/lib/webgl-helpers.js";
import Matrix4 from "@/lib/cuon-matrix";

const glCanvas = ref(null);
const angle = ref(0);
const matrix = ref([]); // 列主序

watch(angle, value => {
  const gl = glCanvas.value.gl;
  if (!gl) return;
  gl.$rotate(value);
  gl.$draw();
});

onMounted(() => {
  const gl = glCanvas.value.gl;

  gl.clearColor(0.0, 0.0, 0.0, 1.0); // 以 RGBA 格式设置颜色，取值范围从 0.0 到 1.0

  // 顶点着色器
  const VSHADER_SOURCE = 
    'attribute vec4 a_Position;\n' +
    'uniform mat4 u_xformMatrix, u_scaleMatrix;\n' +
    'void main() {\n' +
    '  gl_Position = u_scaleMatrix * (u_xformMatrix * a_Position);\n' +
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

  const u_xformMatrix = gl.getUniformLocation(gl.program, 'u_xformMatrix'); // 获取 uniform 变量地址
  if (!u_xformMatrix) {
    console.log("Failed to get the storage location of u_xformMatrix");
    return;
  }

  // u_scaleMatrix
  const u_scaleMatrix = gl.getUniformLocation(gl.program, 'u_scaleMatrix'); // 获取 uniform 变量地址
  if (!u_scaleMatrix) {
    console.log("Failed to get the storage location of u_scaleMatrix");
    return;
  }

  function initVertexBUffers(gl) {
    const vertices = new Float32Array([0, 0, 0.5, 0, 0, 0.5]);
    const n = 3;
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

  const n = initVertexBUffers(gl);

  gl.$rotate = (angle) => {
    const xformMatrix = new Matrix4();
    xformMatrix.setRotate(angle, 0, 0, 1);
    matrix.value = xformMatrix.elements;
    gl.uniformMatrix4fv(u_xformMatrix, false, new Float32Array(matrix.value));
    const s = angle / 360 + 1;
    gl.uniformMatrix4fv(u_scaleMatrix, false, new Float32Array([
      s, 0, 0, 0,
      0, s, 0, 0,
      0, 0, s, 0,
      0, 0, 0, 1
    ]));
  };

  gl.$draw = () => {
    gl.clear(gl.COLOR_BUFFER_BIT);     // 传入 gl.COLOR_BUFFER_BIT 常量，表示要清空的是“颜色缓冲区”
    gl.drawArrays(gl.TRIANGLES, 0, n);
  }

  gl.$rotate(0);
  gl.$draw();
});


</script>

<style lang="scss" scoped></style>