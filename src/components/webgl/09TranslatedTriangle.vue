<template>
  <div>
    <h1>画一个三角形 - 平移</h1>
    <GLCanvas ref="glCanvas" />
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import GLCanvas from "./GLCanvas.vue";
import { initShaders } from "@/lib/webgl-helpers.js";

const glCanvas = ref(null);

onMounted(() => {
  const gl = glCanvas.value.gl;

  gl.clearColor(0.0, 0.0, 0.0, 1.0); // 以 RGBA 格式设置颜色，取值范围从 0.0 到 1.0
  gl.clear(gl.COLOR_BUFFER_BIT);     // 传入 gl.COLOR_BUFFER_BIT 常量，表示要清空的是“颜色缓冲区”

  // 顶点着色器
  const VSHADER_SOURCE = 
    'attribute vec4 a_Position;\n' +
    'uniform vec4 u_Translation;\n' +
    'void main() {\n' +
    '  gl_Position = a_Position + u_Translation;\n' +
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

  const u_Translation = gl.getUniformLocation(gl.program, 'u_Translation'); // 获取 uniform 变量地址
  if (!u_Translation) {
    console.log("Failed to get the storage location of u_Translation");
    return;
  }

  gl.uniform4f(u_Translation, 0.5, 0.5, 0.0, 0.0);

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
  gl.drawArrays(gl.TRIANGLES, 0, n);
  // gl.drawArrays(gl.LINE_STRIP, 0, n);
  // gl.drawArrays(gl.LINE_LOOP, 0, n);
});


</script>

<style lang="scss" scoped></style>