<template>
  <div ref="threeContainer"></div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GUI } from "three/examples/jsm/libs/lil-gui.module.min.js";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";
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
scene.background = new THREE.Color(0xcccccc);
//加载GLTF模型
const gltfLoader = new GLTFLoader();
gltfLoader.load(
  "/model/Duck.glb",
  (glb) => {
    console.log("🚀 ~ glb:", glb);
    scene.add(glb.scene);
  },
  undefined,
  (error) => {
    console.error("An error happened while loading the GLTF model:", error);
  }
);

//加载HDR环境贴图
const rgbeLoader = new RGBELoader();
rgbeLoader.load(
  "/texture/Alex_Hart-Nature_Lab_Bones_2k.hdr",
  (texture) => {
    texture.mapping = THREE.EquirectangularReflectionMapping;
    scene.environment = texture;
  },
  undefined,
  (error) => {
    console.error(
      "An error happened while loading the HDR environment map:",
      error
    );
  }
);

//加载DRACO压缩模型
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath("/draco/");
gltfLoader.setDRACOLoader(dracoLoader);

gltfLoader.load(
  "/model/city.glb",
  (glb) => {
    console.log("🚀 ~ glb:", glb);
    scene.add(glb.scene);
  },
  undefined,
  (error) => {
    console.error("An error happened while loading the GLTF model:", error);
  }
);

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
