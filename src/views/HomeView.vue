<template>
  <div class="home">
    <div ref="container" class="container"></div>
  </div>
</template>

<script setup>
// Vue相关导入
import { onMounted, ref } from 'vue';

// Three.js相关导入
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
// import Parrot from '@/assets/parrot.glb';
import Parrot from '@/assets/donghua.glb';

// 场景相关变量
const container = ref(null);
let camera;
let controls;
let renderer;
let scene;

// 动画相关变量
const mixers = [];
const clock = new THREE.Clock();

// 在组件挂载后初始化Three.js场景
onMounted(() => {
  init();
});

/**
 * 创建相机
 * 设置透视相机的参数并定位
 */
function createCamera() {
  camera = new THREE.PerspectiveCamera(
    35,
    container.value.clientWidth / container.value.clientHeight,
    1,
    1000,
  );
  camera.position.set(-20.5, 21.5, 26.5);
}

/**
 * 创建轨道控制器
 * 配置控制器参数，限制用户交互方式
 */
function createControls() {
  controls = new OrbitControls(camera, container.value);
  controls.enableZoom = false;
  controls.enablePan = false;
  controls.enableDamping = true;
  controls.target.z = 2;
}

/**
 * 创建场景光源
 * 添加半球环境光和平行主光源
 */
function createLights() {
  const ambientLight = new THREE.HemisphereLight(0xddeeff, 0x0f0e0d, 8);

  const mainLight = new THREE.DirectionalLight(0xffffff, 10);
  mainLight.position.set(200, 200, 200);

  scene.add(ambientLight, mainLight);
}

/**
 * 加载3D模型
 * 加载鹦鹉模型并设置动画
 */
function loadModels() {
  const loader = new GLTFLoader();
  const parrotPosition = new THREE.Vector3(0, 0, 2.5);

  // 模型加载成功后的回调函数
  const onLoad = (gltf, position) => {
    const bird = gltf.scene.children[0];
    bird.position.copy(position);

    // 设置动画
    const animation = gltf.animations[0];
    const mixer = new THREE.AnimationMixer(bird);
    mixers.push(mixer);

    const action = mixer.clipAction(animation);
    action.play();

    scene.add(bird);
  };

  // 加载鹦鹉模型
  loader.load(
    Parrot,
    (gltf) => onLoad(gltf, parrotPosition),
    null, // onProgress回调
    null, // onError回调
  );
}

/**
 * 创建渲染器
 * 配置WebGL渲染器并添加到DOM
 */
function createRenderer() {
  renderer = new THREE.WebGLRenderer({
    antialias: true, // 启用抗锯齿
  });

  // 设置渲染器尺寸和质量参数
  renderer.setSize(container.value.clientWidth, container.value.clientHeight);
  renderer.physicallyCorrectLights = true;
  renderer.setPixelRatio(window.devicePixelRatio);

  // 将渲染器的canvas元素添加到页面
  container.value.appendChild(renderer.domElement);
}

/**
 * 更新场景中的动画和控制器
 */
function update() {
  const delta = clock.getDelta();

  // 更新所有动画混合器
  mixers.forEach((mixer) => {
    mixer.update(delta);
  });

  // 更新控制器
  controls.update(delta);
}

/**
 * 渲染场景
 */
function render() {
  renderer.render(scene, camera);
}

/**
 * 窗口大小变化时的处理函数
 * 更新相机和渲染器尺寸
 */
function onWindowResize() {
  camera.aspect = container.value.clientWidth / container.value.clientHeight;
  camera.updateProjectionMatrix(); // 更新相机投影矩阵

  renderer.setSize(container.value.clientWidth, container.value.clientHeight);
}

/**
 * 初始化Three.js场景
 * 创建场景并设置所有必要组件
 */
const init = () => {
  // 初始化场景
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0x8fbcd4); // 设置天蓝色背景

  // 按顺序创建各个组件
  createCamera();
  createControls();
  createLights();
  loadModels();
  createRenderer();

  // 设置动画循环
  renderer.setAnimationLoop(() => {
    update();
    render();
  });

  // 添加窗口大小变化监听
  window.addEventListener('resize', onWindowResize);
};
</script>

<style scoped>
.home {
  height: 100vh;
  width: 100vw;
}

.container {
  height: 100%;
  width: 100%;
}
</style>
