<template>
  <div ref="threeContainer"></div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { ref, onMounted } from "vue";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";
// //引入gui
import { GUI } from "three/examples/jsm/libs/lil-gui.module.min.js";
// import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";

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

//创建环境光
const ambientLight = new THREE.AmbientLight(0xffffff, 2);
scene.add(ambientLight);

const controls = new OrbitControls(camera, renderer.domElement);
const Axes = new THREE.AxesHelper(5);
renderer.setSize(window.innerWidth, window.innerHeight);
camera.position.z = 5;
scene.add(Axes);
const gui = new GUI();

//加载HDR环境贴图
const rgbeLoader = new RGBELoader();
rgbeLoader.load("/texture/Alex_Hart-Nature_Lab_Bones_2k.hdr", (texture) => {
  texture.mapping = THREE.EquirectangularRefractionMapping;
  scene.background = texture;
  scene.environment = texture;
});

//加载清漆贴图
const clearcoatRoughnessTexture = new THREE.TextureLoader().load(
  "/texture/diamond/diamond_emissive.png"
);

//加载法线贴图
const normalTexture = new THREE.TextureLoader().load(
  "/texture/carbon/Carbon_Normal.png"
);

//加载清漆法线贴图
const clearcoatNormalTexture = new THREE.TextureLoader().load(
  "/texture/carbon/Scratched_gold_01_1K_Normal.png"
);
//创建立方体
const geometry = new THREE.BoxGeometry(1, 1, 1);
const planeMaterial = new THREE.MeshPhysicalMaterial({
  color: new THREE.Color(0xffff00),
  clearcoat: 1,
  transparent: true,
  roughness: 0.5,
  // clearcoatRoughnessMap: clearcoatRoughnessTexture,
  // clearcoatRoughness: 0.2,
  normalMap: normalTexture,
  clearcoatNormalMap: clearcoatNormalTexture,
  clearcoatNormalScale: new THREE.Vector2(1, 1),
});

const cube = new THREE.Mesh(geometry, planeMaterial);
scene.add(cube);

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
