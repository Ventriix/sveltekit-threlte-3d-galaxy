<script lang="ts">
  import { T, useFrame } from "@threlte/core";
  import Star, { updateAll as updateAllStars } from "./Star.svelte";
  import Haze, { updateAll as updateAllHaze } from "./Haze.svelte";
  import {
    ARM_X_DIST,
    ARM_X_MEAN,
    ARM_Y_DIST,
    ARM_Y_MEAN,
    ARMS,
    CORE_X_DIST,
    CORE_Y_DIST,
    GALAXY_THICKNESS,
    HAZE_RATIO,
    NUM_STARS,
    OUTER_CORE_X_DIST,
    OUTER_CORE_Y_DIST,
    SPIRAL
  } from "./config";
  import { degToRad } from "three/src/math/MathUtils";

  function gaussianRandom(mean = 0, stdev = 1): number {
    let u = 1 - Math.random();
    let v = Math.random();
    let z = Math.sqrt(-2.0 * Math.log(u)) * Math.cos(2.0 * Math.PI * v);

    return z * stdev + mean;
  }

  export function spiral(
    x: number,
    y: number,
    z: number,
    offset: number
  ): [x: number, y: number, z: number] {
    let r = Math.sqrt(x ** 2 + y ** 2);

    let theta = offset;
    theta += x > 0 ? Math.atan(y / x) : Math.atan(y / x) + Math.PI;
    theta += (r / ARM_X_DIST) * SPIRAL;

    return [r * Math.cos(theta), r * Math.sin(theta), z];
  }

  let objects: { type: "star" | "haze"; position: [x: number, y: number, z: number] }[] = [];

  function generate(numStars: number, type: "star" | "haze") {
    for (let i = 0; i < numStars / 4; i++) {
      objects.push({
        type: type,
        position: [
          gaussianRandom(0, CORE_X_DIST),
          gaussianRandom(0, CORE_Y_DIST),
          gaussianRandom(0, GALAXY_THICKNESS)
        ]
      });
    }

    for (let i = 0; i < numStars / 4; i++) {
      objects.push({
        type: type,
        position: [
          gaussianRandom(0, OUTER_CORE_X_DIST),
          gaussianRandom(0, OUTER_CORE_Y_DIST),
          gaussianRandom(0, GALAXY_THICKNESS)
        ]
      });
    }

    for (let j = 0; j < ARMS; j++) {
      for (let i = 0; i < numStars / 4; i++) {
        objects.push({
          type: type,
          position: spiral(
            gaussianRandom(ARM_X_MEAN, ARM_X_DIST),
            gaussianRandom(ARM_Y_MEAN, ARM_Y_DIST),
            gaussianRandom(0, GALAXY_THICKNESS),
            (j * 2 * Math.PI) / ARMS
          )
        });
      }
    }
  }

  generate(NUM_STARS, "star");
  generate(NUM_STARS * HAZE_RATIO, "haze");

  function updateAll() {
    updateAllStars();
    updateAllHaze();
  }

  let rotation = 0;
  export let rotationSpeed = 1;

  useFrame((_, delta) => {
    updateAll();

    let newRotation = rotation + delta * (rotationSpeed / 50);

    if (newRotation > 360) {
      newRotation = 0;
    }

    rotation = newRotation;
  });
</script>

<T.Scene rotation.x={degToRad(90)} rotation.z={rotation}>
  {#each objects as object}
    {#if object.type === "star"}
      <Star position={object.position} />
    {:else}
      <Haze position={object.position} />
    {/if}
  {/each}
</T.Scene>
