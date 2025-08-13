<template>
  <div ref="threeContainer"></div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { ref, onMounted } from "vue";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
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

  gltfLoader.load(
    // 模型路径
    "./model/Duck.glb",
    // 加载完成回调
    (gltf) => {
      const duckMesh = gltf.scene.getObjectByName("LOD3spShape");
      console.log("🚀 ~ duckMesh:", duckMesh);
      let preMaterial = duckMesh.material;
      console.log("🚀 ~ preMaterial:", preMaterial);

      //创建cap材质
      const phongMaterial = new THREE.MeshPhongMaterial({
        // matcap: new THREE.TextureLoader().load("/texture/matcaps/9.jpg"),
        map: preMaterial.map,
        // color: new THREE.Color(0xffffff),
        envMap: texture,
        refractionRatio: 0.7,
        // reflectivity: 0.9,
        transparent: true,
      });
      duckMesh.material = phongMaterial;
      console.log("🚀 ~ duckMesh:", duckMesh);

      //添加phong材质折射率gui面板
      const phongFolder = gui.addFolder("Phong Material");
      phongFolder
        .add(phongMaterial, "refractionRatio", 0, 1, 0.01)
        .name("折射率");
      phongFolder.add(phongMaterial, "transparent").name("透明度");
      phongFolder.addColor(phongMaterial, "color").name("颜色");
      phongFolder.add(phongMaterial, "reflectivity", 0, 1, 0.01).name("反射率");

      scene.add(gltf.scene);
    }
  );
});

//添加phong材质折射率gui面板

// //加载DRACO压缩模型
const gltfLoader = new GLTFLoader();
// const dracoLoader = new DRACOLoader();
// dracoLoader.setDecoderPath("/draco/");
// gltfLoader.setDRACOLoader(dracoLoader);

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
