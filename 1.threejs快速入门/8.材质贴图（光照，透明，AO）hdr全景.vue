<template>
  <div ref="threeContainer"></div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GUI } from "three/examples/jsm/libs/lil-gui.module.min.js";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";
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

const gui = new GUI();

const controls = new OrbitControls(camera, renderer.domElement);
const Axes = new THREE.AxesHelper(5);
renderer.setSize(window.innerWidth, window.innerHeight);
camera.position.z = 5;
scene.add(Axes);

//创建平面
const planeGeometry = new THREE.PlaneGeometry(1, 1);
const texture = new THREE.TextureLoader().load(
  "/texture/watercover/CityNewYork002_COL_VAR1_1K.png",
  (texture) => {
    //纹理加载完成后执行的回调函数
    console.log("纹理加载完成");
    texture.colorSpace = THREE.SRGBColorSpace; //设置颜色空间为sRGB
    planeMaterial.needsUpdate = true; //更新材质
  }
);

//加载透明图贴图
const alphaTexture = new THREE.TextureLoader().load("/texture/door/alpha.jpg");

//加载光照贴图
const lightTexture = new THREE.TextureLoader().load("/texture/colors.png");

//加载AO贴图
const aoTexture = new THREE.TextureLoader().load(
  "/texture/watercover/CityNewYork002_AO_1K.jpg"
);

// 加载环境全景
const envTexture = new RGBELoader().load(
  "/texture/Alex_Hart-Nature_Lab_Bones_2k.hdr",
  (envMap) => {
    //设置球形映射
    envMap.mapping = THREE.EquirectangularReflectionMapping;
    scene.background = envMap; //设置场景背景
    //设置场景纹理
    scene.environment = envMap; //设置场景环境贴图
    //设置材质的环境贴图
    planeMaterial.envMap = envMap; //设置平面材质的环境贴图
  }
);

const planeMaterial = new THREE.MeshBasicMaterial({
  map: texture,
  // envMap: envTexture, //环境贴图
  // alphaMap: alphaTexture,
  aoMap: aoTexture, //AO贴图
  transparent: true, //开启透明
  // lightMap: lightTexture, //光照贴图
});

gui.add(planeMaterial, "aoMapIntensity").min(0).max(1).name("AO贴图强度");

const plane = new THREE.Mesh(planeGeometry, planeMaterial);

scene.add(plane);

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
