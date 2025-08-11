<template>
  <div ref="threeContainer"></div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GUI } from "three/examples/jsm/libs/lil-gui.module.min.js";
import { ref, onMounted } from "vue";
import * as TWEEN from "three/examples/jsm/libs/tween.module.js";

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

//创建绿色球体
const greenSphereMesh = new THREE.Mesh(
  new THREE.SphereGeometry(1, 32, 32),
  new THREE.MeshBasicMaterial({ color: 0x00ff00 })
);
greenSphereMesh.position.x = -3; // 将绿色球体移动到左侧
scene.add(greenSphereMesh);

//创建球体移动动画
const tweenX = new TWEEN.Tween(greenSphereMesh.position)
  .to({ x: 3 }, 2000) // 移动到x=3的位置
  .easing(TWEEN.Easing.Quadratic.InOut)
  .start(); // 启动动画

//创建y轴移动动画
const tweenY = new TWEEN.Tween(greenSphereMesh.position).to(
  { z: 3 }, // 移动到y=3的位置
  2000
);

const params = {
  stop: () => {
    tweenX.stop(); // 停止x轴动画
    tweenY.stop(); // 停止y轴动画
  },
};
const params2 = {
  start: () => {
    tweenX.start(); // 停止x轴动画
    tweenY.start(); // 停止y轴动画
  },
};

gui.add(params, "stop").name("暂停");
gui.add(params2, "start").name("开始");

tweenX.chain(tweenY); // 将y轴动画链到x轴动画后面

tweenY.chain(tweenX); // 将x轴动画链到y轴动画后面

const animate = () => {
  window.requestAnimationFrame(animate);
  TWEEN.update(); // 更新动画

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
