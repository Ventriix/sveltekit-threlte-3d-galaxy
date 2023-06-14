<script lang="ts">
  import { useThrelte, useRender } from "@threlte/core";
  import { BLOOM_RADIUS, BLOOM_STRENGTH, BLOOM_THRESHOLD } from "$lib/components/galaxy/config";
  import type { Camera } from "three";
  import { RenderPass } from "three/examples/jsm/postprocessing/RenderPass";
  import { UnrealBloomPass } from "three/examples/jsm/postprocessing/UnrealBloomPass";
  import { ACESFilmicToneMapping, Vector2 } from "three";
  import { EffectComposer } from "three/examples/jsm/postprocessing/EffectComposer";
  import { SMAAPass } from "three/examples/jsm/postprocessing/SMAAPass";

  const { scene, renderer, camera } = useThrelte();

  let composer: EffectComposer;

  if (renderer) {
    composer = new EffectComposer(renderer);

    renderer.setClearColor("#000000", 1);
    renderer.toneMapping = ACESFilmicToneMapping;
  }

  function setupEffectComposer(camera: Camera) {
    if (!composer) {
      return;
    }

    composer.passes = [];
    composer.addPass(new RenderPass(scene, camera));
    composer.addPass(
      new UnrealBloomPass(
        new Vector2(window.innerWidth, window.innerHeight),
        BLOOM_STRENGTH,
        BLOOM_RADIUS,
        BLOOM_THRESHOLD
      )
    );
    composer.addPass(new SMAAPass(window.innerWidth, window.innerHeight));
  }

  $: setupEffectComposer($camera);

  useRender((_, delta) => {
    if (composer) {
      composer.render(delta);
    }
  });
</script>
