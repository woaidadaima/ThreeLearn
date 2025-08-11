<template>
  <div ref="threeContainer"></div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { ref, onMounted } from "vue";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";

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
// const rgbeLoader = new RGBELoader();
// rgbeLoader.load("/texture/Alex_Hart-Nature_Lab_Bones_2k.hdr", (texture) => {
//   texture.mapping = THREE.EquirectangularReflectionMapping;
//   scene.background = texture;
//   scene.environment = texture;
// });

//加载DRACO压缩模型
const gltfLoader = new GLTFLoader();
const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath("/draco/");
gltfLoader.setDRACOLoader(dracoLoader);

gltfLoader.load(
  // 模型路径
  "./model/city.glb",
  // 加载完成回调
  (gltf) => {
    // console.log(gltf);
    // scene.add(gltf.scene);

    gltf.scene.traverse((child) => {
      if (child.isMesh) {
        let building = child;
        let geometry = building.geometry;

        // 获取边缘geometry
        let edgesGeometry = new THREE.EdgesGeometry(geometry);
        // // 创建线段材质
        let edgesMaterial = new THREE.LineBasicMaterial({
          color: 0xffffff,
        });

        // 线框geometry
        // let edgesGeometry = new THREE.WireframeGeometry(geometry);
        // 创建线段
        let edges = new THREE.LineSegments(edgesGeometry, edgesMaterial);

        // 更新建筑物世界转换矩阵
        building.updateWorldMatrix(true, true);
        edges.matrix.copy(building.matrixWorld);
        edges.matrix.decompose(edges.position, edges.quaternion, edges.scale);

        // 添加到场景
        scene.add(edges);
      }
    });
  }
);

// gltfLoader.load("/model/building.glb", (glb) => {
//   console.log("🚀 ~ glb:", glb);
//   //通过名称获取mesh
//   const buildingMesh = glb.scene.getObjectByName("Plane045");
//   console.log("🚀 ~ buildingMesh:", buildingMesh);
//   //创建边缘几何体
//   const edgesGeometry = new THREE.EdgesGeometry(buildingMesh.geometry);
//   const edgesMaterial = new THREE.LineBasicMaterial({ color: 0xffffff });
//   const edges = new THREE.LineSegments(edgesGeometry, edgesMaterial);
//   console.log("🚀 ~ edges:", edges);
//   // building.updateWorldMatrix(true, true);
//   edges.matrix.copy(buildingMesh.matrixWorld);
//   edges.matrix.decompose(edges.position, edges.quaternion, edges.scale);

//   //创建线框几何体
//   const wireframeGeometry = new THREE.WireframeGeometry(buildingMesh.geometry);
//   const wireframeMaterial = new THREE.LineBasicMaterial({ color: 0xffffff });
//   const wireframe = new THREE.LineSegments(
//     wireframeGeometry,
//     wireframeMaterial
//   );

//   console.log("🚀 ~ edges:", edges);
//   scene.add(edges);
//   scene.add(wireframe);
//   // scene.add(glb.scene);
// });

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
