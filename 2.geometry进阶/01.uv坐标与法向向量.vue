<template>
  <div ref="threeContainer"></div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { ref, onMounted } from "vue";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";
import { VertexNormalsHelper } from "three/addons/helpers/VertexNormalsHelper.js";

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

//加载HDR环境贴图
const rgbeLoader = new RGBELoader();
rgbeLoader.load("/texture/Alex_Hart-Nature_Lab_Bones_2k.hdr", (texture) => {
  texture.mapping = THREE.EquirectangularReflectionMapping;
  scene.background = texture;
  scene.environment = texture;
  planeMaterial.envMap = texture;
  cubeMaterial.envMap = texture;
});

const texture = new THREE.TextureLoader().load(
  "/texture/uv_grid_opengl.jpg" // 替换为你的纹理路径
);
console.log("🚀 ~ texture:", texture);
//创建三角形
const geometry = new THREE.BufferGeometry();
const vertices = new Float32Array([-1, -1, 0, 1, -1, 0, 1, 1, 0, -1, 1, 0]);
const indecies = new Uint16Array([0, 1, 2, 2, 3, 0]);
const uv = new Float32Array([
  0,
  0, // 第一个顶点的 UV 坐标
  1,
  0, // 第二个顶点的 UV 坐标
  1,
  1, // 第三个顶点的 UV 坐标
  0,
  1, // 第四个顶点的 UV 坐标
]);
//计算法线
// const normal = new Float32Array([
//   0,
//   0,
//   1, // 第一个顶点的法线
//   0,
//   0,
//   1, // 第二个顶点的法线
//   0,
//   0,
//   1, // 第三个顶点的法线
//   0,
//   0,
//   1, // 第四个顶点的法线
// ]);
// geometry.setAttribute("normal", new THREE.BufferAttribute(normal, 3));

geometry.setAttribute("uv", new THREE.BufferAttribute(uv, 2));

geometry.setIndex(new THREE.BufferAttribute(indecies, 1));
geometry.setAttribute("position", new THREE.BufferAttribute(vertices, 3));
//自动计算法向量需要注意如果存在共享顶点，计算方法需要放在共享顶点之后，且无论任何时候都要放在设置位置属性之后
geometry.computeVertexNormals();

console.log("🚀 ~ geometry66:", geometry);
const cubeMaterial = new THREE.MeshBasicMaterial({
  map: texture,
});
const triangle = new THREE.Mesh(geometry, cubeMaterial);
console.log("🚀 ~ triangle:", triangle);
scene.add(triangle);
triangle.position.x = -3;

//创建平面
const planeGeometry = new THREE.PlaneGeometry(2, 2);
const planeMaterial = new THREE.MeshBasicMaterial({
  //   wireframe: true,
  map: texture,
});
const plane = new THREE.Mesh(planeGeometry, planeMaterial);
console.log("🚀 ~ plane:", plane);
scene.add(plane);
plane.position.x = 3;

const helper = new VertexNormalsHelper(plane, 1, 0xff0000);
scene.add(helper);
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
