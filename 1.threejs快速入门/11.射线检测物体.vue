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

const gui = new GUI();

const controls = new OrbitControls(camera, renderer.domElement);
const Axes = new THREE.AxesHelper(5);
renderer.setSize(window.innerWidth, window.innerHeight);
camera.position.z = 5;
scene.add(Axes);

//创建球体
const redSphereMesh = new THREE.Mesh(
  new THREE.SphereGeometry(1, 32, 32),
  new THREE.MeshBasicMaterial({ color: 0xffff00 })
);

scene.add(redSphereMesh);

//创建蓝色球体
const blueSphereMesh = new THREE.Mesh(
  new THREE.SphereGeometry(1, 32, 32),
  new THREE.MeshBasicMaterial({ color: 0x0000ff })
);
blueSphereMesh.position.x = 3; // 将蓝色球体移动到右侧
scene.add(blueSphereMesh);

//创建绿色球体
const greenSphereMesh = new THREE.Mesh(
  new THREE.SphereGeometry(1, 32, 32),
  new THREE.MeshBasicMaterial({ color: 0x00ff00 })
);
greenSphereMesh.position.x = -3; // 将绿色球体移动到左侧
scene.add(greenSphereMesh);

//创建射线源
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();

//鼠标点击事件
window.addEventListener("click", (event) => {
  //将鼠标坐标转换为标准化设备坐标
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

  //更新射线
  raycaster.setFromCamera(mouse, camera);

  //计算与球体的交点
  const intersects = raycaster.intersectObjects([
    redSphereMesh,
    blueSphereMesh,
    greenSphereMesh,
  ]);

  if (intersects.length > 0) {
    const intersectedObject = intersects[0].object;
    console.log("Intersected object:", intersectedObject);

    if (intersectedObject._isSelected) {
      //如果已经选中，则恢复原本颜色
      intersectedObject.material.color.set(intersectedObject._originalColor);
      intersectedObject._isSelected = false;
      return;
    }
    //获取原本的颜色
    intersectedObject._originalColor =
      intersectedObject.material.color.getHex();
    //修改为红色
    intersectedObject.material.color.set(0xff0000);
    intersectedObject._isSelected = true;
  }
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
