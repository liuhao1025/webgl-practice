<template>
  <canvas ref="domRef"></canvas>
</template>
<script setup lang="ts">
import { onMounted, ref } from "vue";
import {
  AmbientLight,
  BoxGeometry,
  Color,
  DirectionalLight,
  Mesh,
  MeshBasicMaterial,
  MeshLambertMaterial,
  PerspectiveCamera,
  PlaneGeometry,
  Scene,
  ShaderMaterial,
  Vector3,
  WebGLRenderer,
} from "three";
import { ArcballControls } from "three/examples/jsm/controls/ArcballControls";

const domRef = ref<HTMLCanvasElement>();

const init = () => {
  const scene = new Scene();

  const camera = new PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );
  camera.position.z = 5;
  scene.add(camera);

  const light = new AmbientLight(0xadadad, 1);
  scene.add(light);
  const directionalLight = new DirectionalLight(0xffffff, 0.7);
  directionalLight.position.set(0, 0, 50);
  scene.add(directionalLight);

  const renderer = new WebGLRenderer({ canvas: domRef.value! });
  renderer.setSize(window.innerWidth, window.innerHeight);

  const geometry = new BoxGeometry(1, 1, 1);
  const material = new MeshLambertMaterial();
  const cube = new Mesh(geometry, material);
  // 自定义 shader
  cube.material.transparent = true;
  cube.material.color = new Color(255, 100, 100);
  cube.material.onBeforeCompile = (shader) => {
    console.log(shader.fragmentShader);
    shader.fragmentShader = shader.fragmentShader.replace(
      `#include <output_fragment>`,
      `
      outgoingLight = vec3(255, 0, 0);
      #include <output_fragment>
    `
    );
  };

  scene.add(cube);

  // add a groud
  {
    const geometry = new PlaneGeometry(5, 5);
    const material = new MeshLambertMaterial({ color: 0x0000ff });
    const ground = new Mesh(geometry, material);
    ground.position.set(0, 0, -1);
    scene.add(ground);
  }

  const controls = new ArcballControls(camera, domRef.value!, scene);
  controls.addEventListener("change", () => {
    renderer.render(scene, camera);
  });
  controls.setGizmosVisible(false);
  controls.update();

  const time = { value: 0 };
  const r = 2;
  const round = (cube: Mesh) => {
    const arc = ((time.value % 360) * (2 * Math.PI)) / 360;
    const x = r * Math.sin(arc);
    const y = r * Math.cos(arc);
    cube.position.x = x;
    cube.position.y = y;
  };

  const kick = () => {
    renderer.render(scene, camera);
    time.value += 1;
    cube.rotation.x += 0.01;
    cube.rotation.y += 0.01;
    cube.rotation.z += 0.01;
    round(cube);
    requestAnimationFrame(kick);
  };

  kick();
};

onMounted(() => {
  init();
});
</script>
