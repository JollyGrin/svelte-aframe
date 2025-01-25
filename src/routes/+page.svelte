<script lang="ts">
import { onMount } from 'svelte';
import 'aframe';
import 'aframe-ar';

// We need to wait for the AFRAME scene to be loaded before we can use it
let sceneLoaded = $state(false);

onMount(() => {
  // Wait for A-Frame to initialize
  const scene = document.querySelector('a-scene');
  if (scene?.hasLoaded) {
    sceneLoaded = true;
  } else {
    scene?.addEventListener('loaded', () => {
      sceneLoaded = true;
    });
  }
});
</script>

<svelte:head>
  <!-- Add viewport meta tag for mobile devices -->
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" />
</svelte:head>

<!-- Main AR Scene -->
<div class="ar-container h-screen w-screen">
  <a-scene 
    embedded 
    arjs="sourceType: webcam; debugUIEnabled: false; detectionMode: mono_and_matrix; matrixCodeType: 3x3;"
  >
    <!-- Define your AR content -->
    <a-marker preset="hiro">
      <!-- A simple red box floating above the marker -->
      <a-box position="0 1 0" material="color: red; opacity: 0.7;" scale="1 1 1"></a-box>
    </a-marker>

    <!-- Camera -->
    <a-entity camera></a-entity>
  </a-scene>

  <!-- Loading indicator -->
  {#if !sceneLoaded}
    <div class="absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 text-white bg-black/50 p-4 rounded">
      Loading AR Scene...
    </div>
  {/if}
</div>

<style lang="postcss">
  /* Remove any margins and make sure scene takes full viewport */
  :global(body) {
    margin: 0;
    overflow: hidden;
  }
</style>
