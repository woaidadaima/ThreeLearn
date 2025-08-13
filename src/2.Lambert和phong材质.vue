<template>
  <div ref="threeContainer"></div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { ref, onMounted } from "vue";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";
//引入gui
import { GUI } from "three/examples/jsm/libs/lil-gui.module.min.js";

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

//创建GUI
const gui = new GUI();

//加载HDR环境贴图
const rgbeLoader = new RGBELoader();
rgbeLoader.load("/texture/Alex_Hart-Nature_Lab_Bones_2k.hdr", (texture) => {
  texture.mapping = THREE.EquirectangularReflectionMapping;
  scene.background = texture;
  scene.environment = texture;
  planeMaterial.envMap = texture; //将环境贴图应用到平面材质上
});

//创建高光贴图
const highTexture = new THREE.TextureLoader().load(
  "/texture/watercover/CityNewYork002_GLOSS_1K.jpg"
);

//创建法线贴图
const normalTexture = new THREE.TextureLoader().load(
  "/texture/watercover/CityNewYork002_NRM_1K.jpg"
);

//创建凹凸贴图
const bumpTexture = new THREE.TextureLoader().load(
  "/texture/watercover/CityNewYork002_DISP_1K.jpg"
);

//创建置换贴图
const displacementTexture = new THREE.TextureLoader().load(
  "/texture/watercover/CityNewYork002_DISP_1K.jpg"
);

//创建ao贴图
const aoTexture = new THREE.TextureLoader().load(
  "/texture/watercover/CityNewYork002_AO_1K.jpg"
);

//创建平面
const planeGeometry = new THREE.PlaneGeometry(1, 1, 200, 200);
const planeMaterial = new THREE.MeshLambertMaterial({
  map: new THREE.TextureLoader().load(
    "/texture/watercover/CityNewYork002_COL_VAR1_1K.png",
    (texture) => {
      console.log("🚀 ~ texture:", texture);
      texture.colorSpace = THREE.SRGBColorSpace;
    }
  ),

  specularMap: highTexture,
  normalMap: normalTexture,
  bumpMap: bumpTexture,
  displacementMap: displacementTexture,
  displacementScale: 0.02, //设置置换贴图的缩放比例
  aoMap: aoTexture,

  //透明度
  transparent: true,
});
const plane = new THREE.Mesh(planeGeometry, planeMaterial);
plane.rotation.x = -Math.PI / 2;
scene.add(plane);

//创建环境光
const ambientLight = new THREE.AmbientLight(0xffffff, 2);
scene.add(ambientLight);

//创建环境光强度gui
gui.add(ambientLight, "intensity", 0, 2, 0.1).name("环境光强度");

//创建点光源
const pointLight = new THREE.PointLight(0xffffff, 1);
pointLight.position.set(0, 3, 0);
scene.add(pointLight);

//创建点光源y轴位置gui
gui.add(pointLight.position, "y", 0, 5, 0.1).name("点光源Y轴位置");

//创建法线贴图是否开启gui
const normalMapEnabled = ref(true);
gui
  .add(normalMapEnabled, "value")
  .name("法线贴图开启")
  .onChange((value) => {
    planeMaterial.normalMap = value ? normalTexture : null;
    planeMaterial.needsUpdate = true; //更新材质
  });

//创建Ao贴图是否开启gui
const aoMapEnabled = ref(true);
gui
  .add(aoMapEnabled, "value")
  .name("AO贴图开启")
  .onChange((value) => {
    planeMaterial.aoMap = value ? aoTexture : null;
    planeMaterial.needsUpdate = true; //更新材质
  });

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
