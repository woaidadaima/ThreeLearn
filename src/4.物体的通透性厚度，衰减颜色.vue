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

//加载厚度贴图
const thicknessTexture = new THREE.TextureLoader().load(
  "/texture/diamond/diamond_emissive.png"
);
//创建立方体
const geometry = new THREE.BoxGeometry(1, 1, 1);
const planeMaterial = new THREE.MeshPhysicalMaterial({
  // color: new THREE.Color(0xffffff),
  transparent: true,
  transmission: 0.99,
  attenuationColor: new THREE.Color(0.8, 0, 0),
  attenuationDistance: 1,
  roughness: 0.05,
  thickness: 2,
  thicknessMap: thicknessTexture,
});

//设置材质厚度gui
gui.add(planeMaterial, "thickness").min(0).max(10).step(0.01).name("thickness");

//设置材质衰减距离gui
gui
  .add(planeMaterial, "attenuationDistance")
  .min(0)
  .max(10)
  .step(0.01)
  .name("attenuationDistance");

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
