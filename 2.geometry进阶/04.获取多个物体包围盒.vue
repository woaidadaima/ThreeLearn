<template>
  <div ref="threeContainer"></div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { ref, onMounted } from "vue";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";

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
camera.position.z = 5;
scene.add(Axes);

//创建红色球体
const redSphereGeometry = new THREE.SphereGeometry(1, 32, 32);
const redSphereMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000 });
const redSphere = new THREE.Mesh(redSphereGeometry, redSphereMaterial);
redSphere.position.set(0, 0, 0);
scene.add(redSphere);

//创建蓝色球体
const blueSphereGeometry = new THREE.SphereGeometry(1, 32, 32);
const blueSphereMaterial = new THREE.MeshBasicMaterial({ color: 0x0000ff });
const blueSphere = new THREE.Mesh(blueSphereGeometry, blueSphereMaterial);
blueSphere.position.set(4, 0, 0);
scene.add(blueSphere);

//创建绿色球体
const greenSphereGeometry = new THREE.SphereGeometry(1, 32, 32);
const greenSphereMaterial = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
const greenSphere = new THREE.Mesh(greenSphereGeometry, greenSphereMaterial);
greenSphere.position.set(-4, 0, 0);
scene.add(greenSphere);

console.log(scene);
const meshGroup = scene.getObjectsByProperty("type", "Mesh");
console.log("🚀 ~ meshGroup:", meshGroup);
let bigBox = new THREE.Box3();
for (let i = 0; i < meshGroup.length; i++) {
  const mesh = meshGroup[i];
  // mesh.geometry.computeBoundingBox();
  // //获取包围盒
  // const box = mesh.geometry.boundingBox;
  // mesh.updateWorldMatrix(true, true);
  // box.applyMatrix4(mesh.matrixWorld);
  //简写形式
  const box = new THREE.Box3().setFromObject(mesh);
  console.log("🚀 ~ box:", box);
  bigBox.union(box);
  scene.add(new THREE.Box3Helper(bigBox, new THREE.Color(0xff0000)));
}

const animate = () => {
  window.requestAnimationFrame(animate);
  controls.update();
  renderer.render(scene, camera);
};

//自适应窗口变化
window.onresize = () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
};

onMounted(() => {
  threeContainer.value.appendChild(renderer.domElement);
  animate();
});
</script>
<style scoped lang="scss"></style>
