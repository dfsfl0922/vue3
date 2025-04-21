<template>
  <div class="company">
    <div class="model" ref="model">
      <canvas ref="canvas"></canvas>
    </div>

    <div class="tips" v-if="tipsShow" @click="intoModel">
      <h3>操作方法</h3>
      <div class="key">
        <div class="key-item">
          <span>W</span>
          <span>前进</span>
        </div>
        <div class="key-box">
          <div class="key-item"><span>A</span><span>向左</span></div>
          <div class="key-item"><span>S</span><span>退后</span></div>
          <div class="key-item"><span>D</span><span>向右</span></div>
        </div>
      </div>
      <h2>点击进入</h2>
    </div>
  </div>
</template>
<script setup>
import * as THREE from "three";
import { PointerLockControls } from "three/examples/jsm/controls/PointerLockControls";
import { FBXLoader } from "three/examples/jsm/loaders/FBxLoader";
import { onMounted, ref, onUnmounted } from "vue";
import { GUI } from "three/examples/jsm/libs/lil-gui.module.min.js";
import { TextGeometry } from "three/examples/jsm/geometries/TextGeometry.js";
import { FontLoader } from "three/examples/jsm/loaders/fontloader.js";
let tipsShow = ref(true);

const gui = new GUI();
const fontloader = new FontLoader();
let camera, scene, renderer, controls;
let canvas = ref(null);
let model = ref(null);
let raycaster = ref(null);
// 控制方向
let moveForward = ref(false);
let moveBackward = ref(false);
let moveLeft = ref(false);
let moveRight = ref(false);
let canJump = ref(false);

let prevTime = performance.now();
const velocity = new THREE.Vector3();
const direction = new THREE.Vector3();
const boundaryMin = new THREE.Vector3(-2600, -100, -2100);
const boundaryMax = new THREE.Vector3(2600, 100, 7000);

onMounted(() => {
  modelInit();
  setCompanyName("智能制造责任有限公司");
});

const intoModel = () => {
  tipsShow.value = false;
  controls.lock();
};

const modelInit = () => {
  // 设置相机
  camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    100,
    100000
  );
  camera.position.y = 250;

  // 设置场景
  scene = new THREE.Scene();
  scene.add(camera); // 改动：将相机添加到场景中
  scene.background = new THREE.Color(0xffffff);

  setLight();

  renderer = new THREE.WebGLRenderer({
    canvas: canvas.value,
    antialias: true,
    alpha: false,
  });
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(window.innerWidth, window.innerHeight);

  // 设置控制器
  controls = new PointerLockControls(camera, canvas.value);

  // 锁定鼠标
  model.value.addEventListener("click", function () {
    controls.lock();
  });

  // 锁定事件
  controls.addEventListener("lock", function () {
    console.log("this is the lock");
  });

  // 解锁的事件
  controls.addEventListener("unlock", function () {
    console.log("this is the unlock");
  });

  // 这里不需要再添加 controls.camera 了
  // scene.add(controls.camera);  // 改动：这一行已去掉

  document.addEventListener("keydown", onKeyDown);
  document.addEventListener("keyup", onKeyUp);
  window.addEventListener("resize", onWindowResize);
  raycaster.value = new THREE.Raycaster(
    new THREE.Vector3(),
    new THREE.Vector3(0, -1, 0),
    0,
    10
  );

  let axesHelper = new THREE.AxesHelper(500); // 坐标轴
  axesHelper.position.set(0, 0, 0);
  scene.add(axesHelper);

  // 模型加载
  modelLoading();

  render();
};

const render = () => {
  renderer.setAnimationLoop(animate);
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap; // 或者其他类型
  renderer.render(scene, camera);
  requestAnimationFrame(render.bind(this));
};

const animate = () => {
  const time = performance.now();

  if (controls.isLocked === true) {
    raycaster._rawValue.ray.origin.copy(controls.camera.position);

    const delta = (time - prevTime) / 68;

    velocity.x -= velocity.x * 10.0 * delta;
    velocity.z -= velocity.z * 10.0 * delta;

    direction.z = Number(moveForward.value) - Number(moveBackward.value);
    direction.x = Number(moveRight.value) - Number(moveLeft.value);
    direction.normalize(); 

    if (moveForward.value || moveBackward.value)
      velocity.z -= direction.z * 800.0 * delta;
    if (moveLeft.value || moveRight.value)
      velocity.x -= direction.x * 800.0 * delta;

    controls.moveRight(-velocity.x * delta);
    controls.moveForward(-velocity.z * delta);
    checkBounds(controls.getObject().position);
    checkTriggerAreas(controls.getObject().position);
  }

  prevTime = time;

  renderer.render(scene, camera);
};

const checkBounds = (position) => {
  position.clamp(boundaryMin, boundaryMax);
};

var pointA = new THREE.Vector3(-2597.56, 0, 2089.37);
var pointB = new THREE.Vector3(-2589.24, 120, 2636.55);

const checkTriggerAreas = (position) => {
  console.log("11223344", position);
  let c = isBetween(position, pointA, pointB);
  console.log(c);
  // if (triggerArea.containsPoint(position)) {
  //   console.log('11223344',position)
  //   // 执行页面跳转逻辑
  //   window.location.href = 'http://example.com/newpage';
  // }
};

function isBetween(modelPosition, pointA, pointB) {
  var minX = Math.min(pointA.x, pointB.x);
  var maxX = Math.max(pointA.x, pointB.x);
  var minY = Math.min(pointA.y, pointB.y);
  var maxY = Math.max(pointA.y, pointB.y);
  var minZ = Math.min(pointA.z, pointB.z);
  var maxZ = Math.max(pointA.z, pointB.z);

  return (
    modelPosition.x >= minX &&
    modelPosition.x <= maxX &&
    modelPosition.y >= minY &&
    modelPosition.y <= maxY &&
    modelPosition.z >= minZ &&
    modelPosition.z <= maxZ
  );
}

const onKeyDown = function (event) {
  switch (event.code) {
    case "KeyW":
      moveForward.value = true;
      break;

    case "KeyA":
      moveLeft.value = true;
      break;

    case "KeyS":
      moveBackward.value = true;
      break;

    case "KeyD":
      moveRight.value = true;
      break;

    case "Space":
      if (canJump.value === true) velocity.y += 350;
      canJump.value = false;
      break;
  }
};

const onKeyUp = function (event) {
  switch (event.code) {
    case "KeyW":
      moveForward.value = false;
      break;

    case "KeyA":
      moveLeft.value = false;
      break;

    case "KeyS":
      moveBackward.value = false;
      break;

    case "KeyD":
      moveRight.value = false;
      break;
  }
};

const modelLoading = () => {
  let fbx = new FBXLoader();
  fbx.load(
    "models/company.fbx",
    (group) => {
      try {
        const scaleValue = 0.5;
        group.scale.set(scaleValue, scaleValue, scaleValue);
        group.position.y = -550;
        camera.position.x = 0;
        camera.position.z = 6500;
        group.rotation.y = Math.PI / 2;
        calcMeshCenter(group);
        scene.add(group);
      } catch {
        // rej('模型加载失败')
        // console.log("模型加载失败");
      }
    },
    (xhr) => {
      // console.log("这里是进度", xhr);
    }
  );
};

const calcMeshCenter = (model) => {
  var box3 = new THREE.Box3();
  box3.expandByObject(model);
  var center = new THREE.Vector3();
  box3.getCenter(center);
  model.position.x = model.position.x - center.x;
  model.position.y = model.position.y - center.y;
  model.position.z = model.position.z - center.z;
};

const setLight = () => {
  const ambientLight = new THREE.AmbientLight(0xffffff, 1); // 减弱一点环境光
  scene.add(ambientLight);
  const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
  directionalLight.position.set(-156, 100, -68); // 调整位置以获得更好的光照效果

  directionalLight.castShadow = true; // 启用阴影
  directionalLight.shadow.mapSize.width = 2048; // 增加阴影贴图大小
  directionalLight.shadow.mapSize.height = 2048;
  directionalLight.shadow.camera.near = 0.5;
  directionalLight.shadow.camera.far = 5000;
  scene.add(directionalLight);
  
  const directionalLight1 = new THREE.DirectionalLight(0xffffff, 1);
  directionalLight1.position.set(2000, 2000, -116); // 调整位置以获得更好的光照效果

  directionalLight1.castShadow = true; // 启用阴影
  directionalLight1.shadow.mapSize.width = 2048; // 增加阴影贴图大小
  directionalLight1.shadow.mapSize.height = 2048;
  directionalLight1.shadow.camera.near = 0.5;
  directionalLight1.shadow.camera.far = 5000;
  scene.add(directionalLight1);
};


const setCompanyName = (companyName) => {
  // 创建文本几何体
 
  var material = new THREE.MeshPhongMaterial({
    color: 0xfff800,
    specular: 0xffff00,
    shininess: 0,
  });
 
  // fontloader.load("font/jtz.json", (font) => {
  //   const textGeometry = new TextGeometry(companyName, {
  //     font, // 字体格式
  //     size: 60, // 大小
  //     height: 0.2, // 高度
  //     curveSegments: 5,
  //     bevelEnabled: true, // 斜切面启动
  //     bevelThickness: 0.03, // 斜面的厚度
  //     bevelSize: 0.02, // 斜切面 会影响计算boundingBox 的值
  //     bevelOffset: 0, // 倒角偏移
  //     bevelSegments: 5, //
  //   });
  //   const textMesh = new THREE.Mesh(textGeometry, material);
  //   textMesh.position.set(0, 300, 3950);
  //   scene.add(textMesh);
  // });
}

const onWindowResize = () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
};
</script>


<style scoped>
.company {
  position: relative;
  width: 100%;
  height: 100%;
}
.model {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}
.tips {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: #fff;
  font-size: 18px;
}
.key {
  display: flex;
  justify-content: center;
  margin-bottom: 10px;
}
.key-box {
  display: flex;
  justify-content: center;
}
.key-item {
  display: flex;
  align-items: center;
  margin: 0 10px;
}
.key-item span {
  margin-right: 10px;
}
</style>
