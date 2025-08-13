<template>
  <div ref="threeContainer"></div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { ref, onMounted } from "vue";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
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

// //加载DRACO压缩模型
const gltfLoader = new GLTFLoader();
// const dracoLoader = new DRACOLoader();
// dracoLoader.setDecoderPath("/draco/");
// gltfLoader.setDRACOLoader(dracoLoader);

gltfLoader.load(
  // 模型路径
  "./model/Duck.glb",
  // 加载完成回调
  (gltf) => {
    console.log("🚀 ~ gltf:", gltf);
    const duckMesh = gltf.scene.getObjectByName("LOD3spShape");
    console.log("🚀 ~ duckMesh:", duckMesh);
    let preMaterial = duckMesh.material;
    console.log("🚀 ~ preMaterial:", preMaterial);

    //创建cap材质
    const capMaterial = new THREE.MeshMatcapMaterial({
      matcap: new THREE.TextureLoader().load("/texture/matcaps/9.jpg"),
      map: preMaterial.map,
    });
    duckMesh.material = capMaterial;
    console.log("🚀 ~ duckMesh:", duckMesh);
    scene.add(gltf.scene);
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
