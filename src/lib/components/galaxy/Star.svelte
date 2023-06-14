<script lang="ts" context="module">
  let updateFunctions: Set<() => void> = new Set();

  export function updateAll() {
    updateFunctions.forEach((update) => update());
  }
</script>

<script lang="ts">
  import { T, useThrelte } from "@threlte/core";
  import { Vector3 } from "three";
  import { useTexture } from "@threlte/extras";
  import {
    STAR_COLORS,
    STAR_MAX_SIZE,
    STAR_MIN_SIZE,
    STAR_SIZES,
    STAR_TYPE_PERCENTAGES
  } from "./config";
  import { onMount } from "svelte";

  const { camera } = useThrelte();

  export let position: [x: number, y: number, z: number] = [0, 0, 0];

  function randomType() {
    let random = Math.random() * 100.0;

    for (let i = 0; i < STAR_TYPE_PERCENTAGES.length; i++) {
      random -= STAR_TYPE_PERCENTAGES[i];

      if (random < 0) {
        return i;
      }
    }

    return 0;
  }

  const type = randomType();

  let scale = 1;

  function updateScale() {
    if (camera.current) {
      scale = Math.min(
        Math.max(
          (new Vector3(position[0], position[1], position[2]).distanceTo(camera.current.position) /
            250) *
            STAR_SIZES[type],
          STAR_MIN_SIZE
        ),
        STAR_MAX_SIZE
      );
    }
  }

  const texture = useTexture("galaxy/sprite120.png");

  onMount(() => {
    updateFunctions.add(updateScale);
    return () => updateFunctions.delete(updateScale);
  });
</script>

{#await texture then value}
  <T.Sprite {position} {scale}>
    <T.SpriteMaterial map={value} color={STAR_COLORS[type]} />
  </T.Sprite>
{/await}
