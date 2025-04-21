<template>
  <div ref="threeContainer" class="three-container"></div>
</template>

<script setup>
import { onMounted, ref } from 'vue';
import * as THREE from 'three';

// 导入 OrbitControls
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

const threeContainer = ref(null);

// Scene 和其他资源初始化
let scene, camera, renderer, controls;

onMounted(() => {
  initScene();
  initCamera();
  initRenderer();
  initControls();
  addLights();
  addCube();
  addGridHelper();
  animate();
});

// 初始化场景
function initScene() {
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0xffffff); // 设置背景颜色
}

// 初始化相机
function initCamera() {
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.z = 5; // 设置相机位置
}

// 初始化渲染器
function initRenderer() {
  renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  threeContainer.value.appendChild(renderer.domElement);
}

// 初始化控制器
function initControls() {
  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true; // 启用阻尼
  controls.dampingFactor = 0.25; // 设置阻尼因子
  controls.screenSpacePanning = false; // 禁止屏幕空间平移
  controls.maxPolarAngle = Math.PI / 2; // 限制上下旋转角度（避免翻转）
}

// 添加简单的立方体
function addCube() {
  const geometry = new THREE.BoxGeometry();
  const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
  const cube = new THREE.Mesh(geometry, material);
  scene.add(cube);

  // 旋转立方体
  function animateCube() {
    cube.rotation.x += 0.01;
    cube.rotation.y += 0.01;
  }

  animateCube();
}

// 添加水平网格
function addGridHelper() {
  const gridHelper = new THREE.GridHelper(10, 10);
  scene.add(gridHelper);
}

// 添加灯光
function addLights() {
  const ambientLight = new THREE.AmbientLight(0x404040, 2); // 环境光
  scene.add(ambientLight);

  const directionalLight = new THREE.DirectionalLight(0xffffff, 1); // 平行光
  directionalLight.position.set(1, 1, 1);
  scene.add(directionalLight);
}

// 动画循环
function animate() {
  requestAnimationFrame(animate);

  controls.update(); // 必须在每次渲染前调用控制器更新

  renderer.render(scene, camera);
}
</script>

<style scoped>
.three-container {
  width: 100%;
  height: 100vh;
  background-color: #ffffff;
}
</style>
