

<template>
  <div>
    <gcanvas v-if="isWeex" ref="canvas_holder" style="width:750;height:750;background-color:rgba(255,0,0,1)"></gcanvas>
    <canvas v-if="!isWeex" ref="canvas_holder" style="width:750px;height:750px;background-color:rgba(255,0,0,1)"></canvas>
    <text class="text">isWeex0:{{isWeex}}</text>
  </div>
</template>
<style>
.text {
  font-size: 20;
}
</style>
<script>
const isWeex = typeof callNative === "function";
import { compile } from "./compile-shader";
import hackLog from "./hack-log";

import { enable, WeexBridge, Image as GImage } from "../src/index.js";



function startRaw(ref) {

  var gl = ref.getContext("webgl");

  //   hackLog(gl);

  const vShader = `
    precision mediump float;  
    attribute vec2 aPosition;
    uniform vec2 offset;
    void main() {
      gl_Position = vec4(aPosition.x, aPosition.y + offset.x, 0, 1);
    }`;
  const fShader = `
    precision mediump float;
    void main() {
        gl_FragColor = vec4(0.5, 0.5, 0.5, 1.0);
    }`;

  const {
    useProgram,
    attributes,
    uniforms,
    fillElements,
    drawElements,
    createElementsBuffer
  } = compile({
    vShader,
    fShader,
    gl
  });

  const eleBuffer1 = createElementsBuffer([0, 1, 2]);
  const eleBuffer2 = createElementsBuffer([0, 1, 2]);
  const posBuffer1 = attributes.aPosition.createBuffer([0, 0, 0, 0.5, 0.5, 0]);
  const posBuffer2 = attributes.aPosition.createBuffer([
    -0.5,
    -0.5,
    -0.5,
    0,
    0,
    -0.5
  ]);

  gl.clearColor(0.0, 0.0, 0.0, 1.0);

  function render() {
    gl.clear(gl.COLOR_BUFFER_BIT);

    uniforms.offset.fill([0.5 * ((Date.now() / 1000) % 1), 0]);

    fillElements(eleBuffer1);
    attributes.aPosition.fill(posBuffer1);
    drawElements(3);

    fillElements(eleBuffer2);
    attributes.aPosition.fill(posBuffer2);
    drawElements(3);
  }

  function tick() {
    render();
    if (ref._swapBuffers) {
      ref._swapBuffers();
    }
    setTimeout(tick, 16);
  }

  tick();
}

export default {
  data() {
    return {
      isWeex: isWeex
    };
  },

  mounted: function() {
    var ref = this.$refs.canvas_holder;

    if (isWeex) {
      ref = enable(ref, {
        debug: true,
        bridge: WeexBridge,
        disableAutoSwap: true
      });
    }

    ref.width = WXEnvironment.deviceWidth;
    ref.height = WXEnvironment.deviceWidth;

    startRaw(ref);
  }
};
</script>