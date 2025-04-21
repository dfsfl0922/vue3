<template>
  <div ref="threeContainer" class="three-container"></div>
</template>

<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue';
import * as THREE from 'three';

// 导入 OrbitControls
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { FBXLoader } from "three/examples/jsm/loaders/FBxLoader";
import {TransformControls} from 'three/examples/jsm/controls/TransformControls.js';

const threeContainer = ref(null);

// Scene 和其他资源初始化
let scene, camera, renderer, controls,transformControls;
let raycaster = new THREE.Raycaster();             // 👉 添加：射线检测器
let pointer = new THREE.Vector2();                 // 👉 添加：鼠标位置

//可以供选择的物体列表
let selectableObjects = [];
let selectedObject = null; // 选中的物体

onMounted(() => {
  initScene();
  initCamera();
  initRenderer();
  initControls();
  addLights();
  addCube();
  modelLoading(); // 加载模型
  addGridHelper();
  animate();
  window.addEventListener('resize', onWindowResize);  // 监听窗口大小变化
  threeContainer.value.addEventListener('pointermove', onPointerMove);
});

onBeforeUnmount(() => {
  window.removeEventListener('resize', onWindowResize);  // 清除事件监听
  threeContainer.value.removeEventListener('pointermove', onPointerMove);
});

const onPointerMove=(event)=>{
  pointer.x = (event.clientX / window.innerWidth) * 2 - 1
  pointer.y = -(event.clientY / window.innerHeight) * 2 + 1
  raycaster.setFromCamera(pointer, camera)
  const intersects=raycaster.intersectObjects(selectableObjects,false)
  if(intersects.length>0){
    const object=intersects[0].object
    if(object!=transformControls.object){
      transformControls.attach(object)
      controls.enabled=false
  }
  }
}

// 初始化场景
function initScene() {
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0xffffff); // 设置背景颜色
}

// 初始化相机
function initCamera() {
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 99000);
  camera.position.z = 10; // 设置相机位置
  camera.position.y = 8; // 设置相机高度
  camera.position.x = 10; // 设置相机水平位置
}

// 初始化渲染器
function initRenderer() {
  renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  threeContainer.value.appendChild(renderer.domElement);
}

// 初始化控制器
const initControls=()=> {
  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true; // 启用阻尼
  controls.dampingFactor = 0.25; // 设置阻尼因子
  controls.screenSpacePanning = false; // 禁止屏幕空间平移
  controls.maxPolarAngle = Math.PI / 2; // 限制上下旋转角度（避免翻转）

  transformControls = new TransformControls(camera, renderer.domElement);
  scene.add(transformControls.getHelper());

}
const modelLoading = () => {
  let fbx = new FBXLoader();
  fbx.load(
    "models/company2.fbx",
    (group) => {
      try {

        scene.add(group);
      } catch {
        console.log("模型加载失败", group);
      }
    },
    (xhr) => {
      // console.log("这里是进度", xhr);
    }
  );
}
const calcMeshCenter = (model) => {
  var box3 = new THREE.Box3();
  box3.expandByObject(model);
  var center = new THREE.Vector3();
  box3.getCenter(center);
  model.position.x = model.position.x - center.x;
  model.position.y = model.position.y - center.y;
  model.position.z = model.position.z - center.z;
}

// 添加简单的立方体
function addCube() {
  const geometry = new THREE.BoxGeometry();
  const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
  const cube = new THREE.Mesh(geometry, material);
  selectableObjects.push(cube); // 将立方体添加到可选择的物体列表

  scene.add(cube);
  // 多添加几个随机物体到场景中，供选择
  
  

  
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

// 窗口大小变化时更新相机和渲染器
function onWindowResize() {
  // 更新渲染器大小
  renderer.setSize(window.innerWidth, window.innerHeight);
  
  // 更新相机的纵横比
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix(); // 更新投影矩阵
}
</script>

<style scoped>
.three-container {
  width: 100%;
  height: 100vh;
  background-color: #ffffff;
}
</style>
