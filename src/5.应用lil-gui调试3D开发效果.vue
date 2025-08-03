<template>
  <div ref="threeContainer"></div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GUI } from "three/examples/jsm/libs/lil-gui.module.min.js";
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
const fatherMaterial = new THREE.MeshBasicMaterial({
  color: 0xff0000,
  wireframe: true,
});

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

const eventObj = {
  fullScreen: () => {
    document.body.requestFullscreen();
  },
  exitFullScreen: () => {
    console.log("退出全屏");
    document.exitFullscreen();
  },
};

const cubeColorParams = {
  cubeColor: "#00ff00",
};

//创建GUI
const gui = new GUI();

gui.add(eventObj, "fullScreen").name("全屏");
gui.add(eventObj, "exitFullScreen").name("退出全屏");
gui.add(fatherMaterial, "wireframe").name("父元素线框模式");
// gui.add(cube.position, "x", -5, 5).name("x轴坐标");
gui
  .addColor(cubeColorParams, "cubeColor")
  .onChange((val) => {
    cube.material.color.set(val);
  })
  .name("立方体颜色");
const folder = gui.addFolder("cube");
folder
  .add(cube.position, "x")
  .min(-10)
  .max(10)
  .step(1)
  .onChange((val) => {
    console.log(`x轴 坐标改变为${val}`);
  })
  .name("x轴坐标");
folder
  .add(cube.position, "y")
  .min(-10)
  .max(10)
  .step(1)
  .onFinishChange((val) => {
    console.log(`y轴 坐标改变为${val}`);
  })
  .name("y轴坐标");
folder.add(cube.position, "z").min(-10).max(10).step(1).name("z轴坐标");

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
