<template>
  <div ref="threeContainer"></div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { ref, onMounted } from "vue";

const threeContainer = ref(null);
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
const renderer = new THREE.WebGLRenderer({
  //抗锯齿
  antialias: true,
});

const controls = new OrbitControls(camera, renderer.domElement);
const Axes = new THREE.AxesHelper(5);
renderer.setSize(window.innerWidth, window.innerHeight);

const box = new THREE.BoxGeometry(1, 1, 1);
const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
const fatherMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000 });

const cube = new THREE.Mesh(box, material);
const fatherCube = new THREE.Mesh(box, fatherMaterial);
fatherCube.add(cube);

scene.add(fatherCube);
scene.add(Axes);
// fatherCube.position.x = -2;
fatherCube.position.set(-3, 0, 0);

cube.position.x = 3;
cube.scale.set(2, 2, 2);
fatherCube.scale.set(2, 2, 2);
cube.rotation.set(Math.PI / 4, 0, 0);
fatherCube.rotation.set(Math.PI / 4, 0, 0);
camera.position.z = 5;
const animate = () => {
  window.requestAnimationFrame(animate);
  controls.update();
  // cube.rotation.x += 0.001;
  // cube.rotation.y += 0.001;
  renderer.render(scene, camera);
};

onMounted(() => {
  threeContainer.value.appendChild(renderer.domElement);
  animate();
});
</script>
<style scoped lang="scss">
.threeContainer {
  width: 100%;
  height: 100%;
}
</style>
