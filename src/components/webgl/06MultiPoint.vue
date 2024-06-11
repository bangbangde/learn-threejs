<template>
  <div style="height: 100%; overflow: auto;">
    <h1>使用 buffer 绘制多点</h1>
    <GLCanvas ref="glCanvas" />
    <p>
      - gl.createBuffer
      - gl.bindBuffer
      - gl.bufferData
      - gl.vertexAttribPointer
      - gl.enableVertexAttribArray
    </p>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import GLCanvas from "./GLCanvas.vue";
import { initShaders } from "@/lib/webgl-helpers.js";

const glCanvas = ref(null);


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

  gl.clear(gl.COLOR_BUFFER_BIT);

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
  gl.drawArrays(gl.POINTS, 0, n)
}

onMounted(() => {
  initGL();
})
</script>

<style lang="scss" scoped></style>