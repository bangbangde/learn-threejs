<template>
  <div class="TheSceen" ref="rootRef"></div>
</template>

<script setup>
import * as THREE from 'three';
import { onMounted, ref } from "vue";

const rootRef = ref(null);

function init () {
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

  const renderer = new THREE.WebGLRenderer();
  renderer.setSize(rootRef.value.offsetWidth, rootRef.value.offsetHeight);
  rootRef.value.appendChild(renderer.domElement);

  const geometry = new THREE.BoxGeometry(1, 1, 1);
  const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
  const cube = new THREE.Mesh(geometry, material);
  scene.add(cube);

  camera.position.z = 5;
  function animate () {
    cube.rotation.x += 0.01;
    cube.rotation.y += 0.01;
    renderer.render(scene, camera);
  }
  renderer.setAnimationLoop(animate);
}

onMounted(() => {
  init();
})
</script>

<style lang="scss" scoped>
.TheSceen {
  width: 100%;
  height: 100%;
}
</style>