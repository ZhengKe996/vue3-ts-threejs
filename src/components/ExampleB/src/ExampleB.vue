<template>
  <div ref="threeRef"></div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import {
  AxesHelper,
  BoxGeometry,
  Color,
  Mesh,
  MeshLambertMaterial,
  PerspectiveCamera,
  PlaneGeometry,
  Scene,
  SphereGeometry,
  SpotLight,
  WebGLRenderer,
} from "three";

const threeRef = ref();

const init = (): void => {
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
  scene.add(spotLight);

  // 渲染器
  const renderer = new WebGLRenderer();
  renderer.setClearColor(new Color(0xeeeeee));
  renderer.setSize(window.innerWidth, window.innerHeight);
  // 是否渲染阴影
  renderer.shadowMap.enabled = true;
  // 相机
  const camera = new PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.x = -30;
  camera.position.y = 40;
  camera.position.z = 30;
  camera.lookAt(scene.position);
  threeRef.value.appendChild(renderer.domElement);

  renderer.render(scene, camera);
};
onMounted(() => {
  init();
});
</script>

<style scoped></style>
