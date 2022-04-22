<template>
  <div ref="statesDivRef"></div>
  <div ref="threeRef" :style="{ position: 'absolute', left: 0, top: 0 }"></div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import {
  AxesHelper,
  BoxGeometry,
  Color,
  Mesh,
  MeshLambertMaterial,
  PCFSoftShadowMap,
  PerspectiveCamera,
  PlaneGeometry,
  Scene,
  SphereGeometry,
  SpotLight,
  WebGLRenderer,
} from "three";
import States from "stats.js";
import * as dat from "dat.gui";

const threeRef = ref<HTMLDivElement>();
const statesDivRef = ref<HTMLDivElement>();
const statesRef = ref<States>();
const initState = () => {
  statesRef.value = new States();
  statesRef.value.showPanel(0);
  if (statesRef.value) {
    statesDivRef.value?.appendChild(statesRef.value.dom);
  }
};

const init = (): void => {
  initState();
  const scene = new Scene();

  // 坐标系
  const axes = new AxesHelper(20);
  scene.add(axes);

  const planeGeometry = new PlaneGeometry(50, 20);
  const planeMaterial = new MeshLambertMaterial({ color: 0xffffff });
  const plane = new Mesh(planeGeometry, planeMaterial);
  // 是否允许投射阴影
  plane.receiveShadow = true;

  // 旋转 -90 度
  plane.rotation.x = Math.PI / -2;
  plane.position.x = 15;
  plane.position.y = 0;
  plane.position.z = 0;
  scene.add(plane);

  // 添加立方体
  const cubeGeometry = new BoxGeometry(4, 4, 4);
  const cubeMaterial = new MeshLambertMaterial({ color: 0xff0000 });
  const cube = new Mesh(cubeGeometry, cubeMaterial);
  cube.castShadow = true;
  cube.position.x = 2;
  cube.position.y = 2;
  cube.position.z = 2;
  scene.add(cube);

  // 添加球
  const sphereGeometry = new SphereGeometry(4, 20, 20);
  const sphereMaterial = new MeshLambertMaterial({ color: 0x7777ff });
  const sphere = new Mesh(sphereGeometry, sphereMaterial);
  sphere.castShadow = true;
  sphere.position.x = 15;
  sphere.position.y = 4;
  sphere.position.z = 2;
  scene.add(sphere);

  // 添加灯光
  const spotLight = new SpotLight(0xffffff);
  spotLight.position.set(-40, 60, -10);
  spotLight.castShadow = true;
  spotLight.shadow.mapSize.width = 2048;
  spotLight.shadow.mapSize.height = 2048;
  scene.add(spotLight);

  // 渲染器
  const renderer = new WebGLRenderer();
  renderer.setClearColor(new Color(0xeeeeee));
  renderer.setSize(window.innerWidth, window.innerHeight);

  // 是否渲染阴影
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = PCFSoftShadowMap;

  // 相机
  const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.x = -30;
  camera.position.y = 40;
  camera.position.z = 30;
  camera.lookAt(scene.position);
  threeRef.value?.appendChild(renderer.domElement);

  const controls = {
    rotationSpeed: 0.02,
    bouncingSpeed: 0.03,
    numberOfObjects: scene.children.length,
    addCube: function () {
      const cubeGeometry = new BoxGeometry(4, 4, 4);
      const cubeMaterial = new MeshLambertMaterial({ color: 0xff0000 });
      const cube = new Mesh(cubeGeometry, cubeMaterial);
      cube.castShadow = true;
      cube.name = `cube-${scene.children.length}`;
      cube.position.x = Math.round(Math.random() * planeGeometry.parameters.width);
      cube.position.y = Math.round(Math.random() * 5);
      cube.position.z = -20 + Math.round(Math.random() * planeGeometry.parameters.height);
      scene.add(cube);
      this.numberOfObjects = scene.children.length;
    },
    removeCube: function () {
      const meshList = scene.children.filter((e) => e.name.startsWith("cube-"));
      scene.remove(meshList[meshList.length - 1]);
      this.numberOfObjects = scene.children.length;
    },
  };
  const gui = new dat.GUI();
  gui.add(controls, "rotationSpeed", 0, 0.5);
  gui.add(controls, "bouncingSpeed", 0, 0.5);
  gui.add(controls, "numberOfObjects").listen();
  gui.add(controls, "addCube");
  gui.add(controls, "removeCube");

  let step = 0;
  const renderScene = () => {
    statesRef.value?.begin();
    // 添加新模型时 有用！
    scene.traverse((e) => {
      if (e instanceof Mesh && e != plane) {
        e.rotation.x += controls.rotationSpeed;
        e.rotation.y += controls.rotationSpeed;
        e.rotation.z += controls.rotationSpeed;
      }
    });
    cube.rotation.x += controls.rotationSpeed;
    cube.rotation.y += controls.rotationSpeed;
    cube.rotation.z += controls.rotationSpeed;

    step += controls.bouncingSpeed;
    sphere.position.x = 20 + 10 * Math.cos(step);
    sphere.position.y = 2 + 10 * Math.abs(Math.sin(step));

    statesRef.value?.end();
    requestAnimationFrame(renderScene);
    renderer.render(scene, camera);
  };
  renderScene();
  window.onresize = () => {
    renderer.setSize(window.innerWidth, window.innerHeight);
    camera.aspect = window.innerWidth / window.innerHeight;
  };
};
const clearGui = () => {
  const dg = document.querySelector(".dg");
  if (dg) dg.innerHTML = "";
};
onMounted(() => {
  clearGui();
  init();
});
</script>

<style scoped></style>
