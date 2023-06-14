<script lang="ts">
  import { Canvas, T } from "@threlte/core";
  import Scene from "./Scene.svelte";
  import { Environment, OrbitControls } from "@threlte/extras";
  import Renderer from "./Renderer.svelte";
</script>

<div class="container">
  <Canvas
    rendererParameters={{
      powerPreference: "high-performance",
      antialias: false,
      stencil: false,
      depth: false
    }}
  >
    <T.PerspectiveCamera
      makeDefault
      position={[500, 200, 0]}
      fov={60}
      near={0.1}
      far={5000000}
      on:create={({ ref }) => {
        ref.lookAt(0, 0, 0);
      }}
    >
      <OrbitControls
        enableZoom
        enableDamping
        screenSpacePanning={false}
        minDistance={16}
        maxDistance={16384}
        dampingFactor={0.05}
      />
    </T.PerspectiveCamera>
    <Environment
      path="/galaxy/skybox/"
      files={["front.png", "back.png", "right.png", "left.png", "top.png", "bottom.png"]}
      isBackground
    />
    <Renderer />
    <Scene />
  </Canvas>
</div>

<style>
  .container {
    height: 100vh;
    width: 100vw;
  }
</style>
