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

//加载HDR环境贴图
const rgbeLoader = new RGBELoader();
rgbeLoader.load("/texture/Alex_Hart-Nature_Lab_Bones_2k.hdr", (texture) => {
  texture.mapping = THREE.EquirectangularReflectionMapping;
  scene.background = texture;
  scene.environment = texture;
});

//创建包围盒
const gltfLoader = new GLTFLoader();
gltfLoader.load("/model/Duck.glb", (glb) => {
  console.log("🚀 ~ glb:", glb.scene);
  //通过名称获取mesh
  const duckMesh = glb.scene.getObjectByName("LOD3spShape");
  //更新世界矩阵

  const box = new THREE.Box3();
  //此方法会自动计算包围盒还有包围盒的世界矩阵
  // box.setFromObject(duckMesh);

  //获取包围盒中心点
  const center = box.getCenter(new THREE.Vector3());
  console.log("🚀 ~ center:", center);
  duckMesh.geometry.center();
  //更新世界矩阵
  box.copy(duckMesh.geometry.boundingBox);
  console.log(
    "🚀 ~ box:",
    box,
    duckMesh.geometry.boundingBox,
    duckMesh.matrixWorld
  );

  box.applyMatrix4(duckMesh.matrixWorld);

  console.log("🚀 ~ duckMesh:", duckMesh);

  //设置包围盒中心点到原点
  scene.add(new THREE.Box3Helper(box, new THREE.Color(0xff0000)));
  scene.add(glb.scene);

  //创建包围球
  const sphere = new THREE.SphereGeometry(
    duckMesh.geometry.boundingSphere.radius,
    16,
    16
  );
  const sphereMaterial = new THREE.MeshBasicMaterial({
    color: 0x00ff00,
    wireframe: true,
  });
  const sphereMesh = new THREE.Mesh(sphere, sphereMaterial);
  //更新世界矩阵
  sphereMesh.applyMatrix4(duckMesh.matrixWorld);
  sphereMesh.position.copy(duckMesh.geometry.boundingSphere.center);
  scene.add(sphereMesh);
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
