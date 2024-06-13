<template>
  <div>
    <h1>画一个三角形 - 动画</h1>
    <GLCanvas ref="glCanvas" />
    <div>
      <label>
        speed:
        <input type="range" v-model="speed" min="0.1" max="10" step="0.1">
      </label>
      <p>{{ speed * 1000 }} 度/秒</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import GLCanvas from "./GLCanvas.vue";
import { initShaders } from "@/lib/webgl-helpers.js";
import Matrix4 from "@/lib/cuon-matrix";

const glCanvas = ref(null);
const speed = ref(0.1);

onMounted(() => {
  const gl = glCanvas.value.gl;

  // 顶点着色器
  const VSHADER_SOURCE = 
    'attribute vec4 a_Position;\n' +
    'uniform mat4 u_xformMatrix;\n' +
    'void main() {\n' +
    '  gl_Position = u_xformMatrix * a_Position;\n' +
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

  gl.clearColor(0.0, 0.0, 0.0, 1.0); // 以 RGBA 格式设置颜色，取值范围从 0.0 到 1.0
  gl.clear(gl.COLOR_BUFFER_BIT);     // 传入 gl.COLOR_BUFFER_BIT 常量，表示要清空的是“颜色缓冲区”

  let currentAngle;
  let zero;

  function tick(timeStamp) {
    if (!zero) {
      zero = timeStamp;
      currentAngle = 0;
    } else {
      currentAngle = ((timeStamp - zero) * speed.value) % 360; 
    }
    const modelMatrix = new Matrix4();
    modelMatrix.setRotate(currentAngle, 0, 0, 1);
    // modelMatrix.translate(0.2, 0, 0);
    gl.uniformMatrix4fv(u_xformMatrix, false, modelMatrix.elements);

    gl.clear(gl.COLOR_BUFFER_BIT);     // 传入 gl.COLOR_BUFFER_BIT 常量，表示要清空的是“颜色缓冲区”
    gl.drawArrays(gl.TRIANGLES, 0, n);

    requestAnimationFrame(tick);
  }

  tick();
});


</script>

<style lang="scss" scoped></style>