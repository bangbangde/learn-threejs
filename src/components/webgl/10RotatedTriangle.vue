<template>
  <div>
    <h1>画一个三角形 - 旋转</h1>
    <GLCanvas ref="glCanvas" />
    <input type="range" v-model="angle" min="0" max="360" step="1">
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from "vue";
import GLCanvas from "./GLCanvas.vue";
import { initShaders } from "@/lib/webgl-helpers.js";

const glCanvas = ref(null);
const angle = ref(0);

watch(angle, value => {
  const gl = glCanvas.value.gl;
  if (!gl) return;
  gl.$rotate(value);
  gl.$draw();
})

onMounted(() => {
  const gl = glCanvas.value.gl;

  gl.clearColor(0.0, 0.0, 0.0, 1.0); // 以 RGBA 格式设置颜色，取值范围从 0.0 到 1.0

  // 顶点着色器
  const VSHADER_SOURCE = 
    'attribute vec4 a_Position;\n' +
    'uniform float u_SinB, u_CosB;\n' +
    'void main() {\n' +
    '  gl_Position.x = a_Position.x * u_CosB - a_Position.y * u_SinB;\n' +
    '  gl_Position.y = a_Position.x * u_SinB + a_Position.y * u_CosB;\n' +
    '  gl_Position.z = a_Position.z;\n' +
    '  gl_Position.w = 1.0;\n' +
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

  const u_SinB = gl.getUniformLocation(gl.program, 'u_SinB'); // 获取 uniform 变量地址
  if (!u_SinB) {
    console.log("Failed to get the storage location of u_SinB");
    return;
  }
  const u_CosB = gl.getUniformLocation(gl.program, 'u_CosB'); // 获取 uniform 变量地址
  if (!u_CosB) {
    console.log("Failed to get the storage location of u_CosB");
    return;
  }

  function initVertexBUffers(gl) {
    const vertices = new Float32Array([0, 0.5, -0.5, -0.5, 0.5, -0.5]);
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

  gl.$rotate = (angle => {
    const radian = Math.PI * angle / 180;
    const sinB = Math.sin(radian);
    const cosB = Math.cos(radian);

    gl.uniform1f(u_SinB, sinB);
    gl.uniform1f(u_CosB, cosB);
  });

  gl.$draw = () => {
    gl.clear(gl.COLOR_BUFFER_BIT);     // 传入 gl.COLOR_BUFFER_BIT 常量，表示要清空的是“颜色缓冲区”
    gl.drawArrays(gl.TRIANGLES, 0, n);
  }

  gl.$rotate(0);
  gl.$draw();
});


</script>

<style lang="scss" scoped></style>