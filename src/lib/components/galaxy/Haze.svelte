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
  import { HAZE_MAX_SIZE, HAZE_MIN_OPACITY, HAZE_MIN_SIZE, HAZE_OPACITY } from "./config";
  import { onMount } from "svelte";

  const { camera } = useThrelte();

  export let position: [x: number, y: number, z: number] = [0, 0, 0];

  let opacity = HAZE_OPACITY;

  function updateOpacity() {
    if (camera.current) {
      opacity = Math.min(
        Math.max(
          HAZE_OPACITY *
            Math.pow(
              new Vector3(position[0], position[1], position[2]).distanceTo(
                camera.current.position
              ) /
                250 /
                2.5,
              2
            ),
          HAZE_MIN_OPACITY
        ),
        HAZE_OPACITY
      );
    }
  }

  const texture = useTexture("galaxy/feathered60.png");

  onMount(() => {
    updateFunctions.add(updateOpacity);
    return () => updateFunctions.delete(updateOpacity);
  });
</script>

{#await texture then value}
  <T.Sprite
    {position}
    scale={Math.min(Math.max(HAZE_MAX_SIZE * Math.random(), HAZE_MIN_SIZE), HAZE_MAX_SIZE)}
  >
    <T.SpriteMaterial map={value} color="#0082FF" {opacity} depthTest={false} depthWrite={false} />
  </T.Sprite>
{/await}
