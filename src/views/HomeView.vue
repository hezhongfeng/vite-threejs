<template>
  <div class="home">
    <div class="container"></div>
  </div>
</template>

<script setup>
import { onMounted } from 'vue';

onMounted(() => {
  init();
});
import * as THREE from 'three';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import Parrot from '@/assets/parrot.glb';

let container;
let camera;
let controls;
let renderer;
let scene;

const mixers = [];
const clock = new THREE.Clock();

function createCamera() {
  camera = new THREE.PerspectiveCamera(35, container.clientWidth / container.clientHeight, 1, 1000);
  camera.position.set(-100.5, 101.5, 166.5);
}

function createControls() {
  controls = new OrbitControls(camera, container);
  controls.enableZoom = false;
  controls.enablePan = false;
  controls.enableDamping = true;
  controls.target.z = 2;
}

function createLights() {
  const ambientLight = new THREE.HemisphereLight(0xddeeff, 0x0f0e0d, 8);

  const mainLight = new THREE.DirectionalLight(0xffffff, 3);
  mainLight.position.set(10, 10, 10);

  scene.add(ambientLight, mainLight);
}

function loadModels() {
  const onLoad = (gltf, position) => {
    const bird = gltf.scene.children[0];
    bird.position.copy(position);

    const animation = gltf.animations[0];

    const mixer = new THREE.AnimationMixer(bird);
    mixers.push(mixer);

    const action = mixer.clipAction(animation);
    action.play();

    scene.add(bird);
  };

  const loader = new GLTFLoader();

  const parrotPosition = new THREE.Vector3(0, 0, 2.5);
  loader.load(Parrot, (gltf) => onLoad(gltf, parrotPosition), null, null);
}

function createRenderer() {
  // create a WebGLRenderer and set its width and height
  renderer = new THREE.WebGLRenderer({
    antialias: true,
  });
  renderer.setSize(container.clientWidth, container.clientHeight);

  renderer.physicallyCorrectLights = true;
  renderer.setPixelRatio(window.devicePixelRatio);

  // add the automatically created <canvas> element to the page
  container.appendChild(renderer.domElement);
}

function update() {
  // controls.update();
  const delta = clock.getDelta();

  mixers.forEach((mixer) => {
    mixer.update(delta);
  });
  controls.update(delta);
}

function render() {
  renderer.render(scene, camera);
}

function onWindowResize() {
  camera.aspect = container.clientWidth / container.clientHeight;

  // update the camera's frustum
  camera.updateProjectionMatrix();

  renderer.setSize(container.clientWidth, container.clientHeight);
}

const init = () => {
  container = document.querySelector('.container');

  scene = new THREE.Scene();
  scene.background = new THREE.Color(0x8fbcd4);

  createCamera();
  createControls();
  createLights();
  loadModels();
  createRenderer();

  renderer.setAnimationLoop(() => {
    update();
    render();
  });

  window.addEventListener('resize', onWindowResize);
};
</script>

<style>
.home {
  height: 100vh;
  width: 100vw;
  .container {
    height: 100%;
    width: 100%;
  }
}
</style>
