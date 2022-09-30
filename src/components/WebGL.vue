<template>
  <canvas ref="domRef" style="width: 100%; height: 100%"></canvas>
</template>
<script setup lang="ts">
import { Shader } from "three";
import { onMounted, ref } from "vue";
const domRef = ref<HTMLCanvasElement>();

const vertex = `
  attribute vec4 a_position;
  varying vec4 vPosition;
  void main() {
    vPosition = a_position;
    gl_Position = a_position;
  }
`;

const fragment = `
  precision mediump float;
  varying vec4 vPosition;
  uniform float time;
  vec4 distColor;
  void main() {
    float x1 = vPosition.x / 0.5;
    if (x1 > 0.5) {
      distColor = vec4(1, x1, 0, 1);
    } else {
      distColor = vec4(1, 0, 0, 1);
    }
    gl_FragColor = distColor;
  }
`;

const createShader = (
  gl: WebGLRenderingContext,
  type: number,
  source: string
) => {
  const shader = gl.createShader(type);
  if (shader) {
    gl.shaderSource(shader, source);
    gl.compileShader(shader);
    const flag = gl.getShaderParameter(shader, gl.COMPILE_STATUS);
    if (flag) {
      return shader;
    }
    console.log(`failed to create shader: `, gl.getShaderInfoLog(shader));
    gl.deleteShader(shader);
    return null;
  } else {
    return null;
  }
};

const initCanvas = () => {
  const canvas = domRef.value!;
  const gl: WebGLRenderingContext | null = canvas.getContext("webgl");
  if (gl) {
    const vertexShader = createShader(gl, gl.VERTEX_SHADER, vertex);
    const fragmentShader = createShader(gl, gl.FRAGMENT_SHADER, fragment);
    const program = gl.createProgram();

    if (vertexShader && fragmentShader && program) {
      gl.attachShader(program, vertexShader);
      gl.attachShader(program, fragmentShader);
      gl.linkProgram(program);

      const positionAttributeLocation = gl.getAttribLocation(
        program,
        "a_position"
      );
      const positionBuffer = gl.createBuffer();
      gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);
      const positions = [0, 0, 0, 0.5, 0.5, 0, 0.5, 0, 0.5, 0.5, 0, 0.5];
      gl.bufferData(
        gl.ARRAY_BUFFER,
        new Float32Array(positions),
        gl.STATIC_DRAW
      );

      canvas.width = canvas.clientWidth;
      canvas.height = canvas.clientHeight;
      gl.viewport(0, 0, canvas.width, canvas.height);
      gl.clearColor(0, 0, 0, 0);
      gl.clear(gl.COLOR_BUFFER_BIT);

      gl.useProgram(program);
      gl.enableVertexAttribArray(positionAttributeLocation);

      gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);
      gl.vertexAttribPointer(
        positionAttributeLocation,
        2,
        gl.FLOAT,
        false,
        0,
        0
      );

      gl.drawArrays(gl.TRIANGLES, 0, 6);
      console.log(`draw success.`);
    }
  }
};

onMounted(() => {
  initCanvas();
});
</script>
