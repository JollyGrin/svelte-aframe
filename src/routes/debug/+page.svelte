<script lang="ts">
	import { onMount } from 'svelte';

	// onMount(() => {
	// 	// Check if the scripts have been loaded
	// 	const aframeScript = document.querySelector(
	// 		'script[src="https://aframe.io/releases/0.8.0/aframe.min.js"]'
	// 	);
	// 	const arjsScript = document.querySelector(
	// 		'script[src="https://cdn.rawgit.com/jeromeetienne/AR.js/1.6.0/aframe/build/aframe-ar.js"]'
	// 	);

	// 	if (aframeScript && arjsScript) {
	// 		console.log('Both scripts loaded successfully');
	// 	} else {
	// 		console.error('Scripts not loaded');
	// 	}
	// });
</script>

<svelte:head>
	<script src="https://aframe.io/releases/0.8.0/aframe.min.js"></script>
	<script src="https://cdn.rawgit.com/jeromeetienne/AR.js/1.6.0/aframe/build/aframe-ar.js"></script>
	<meta
		name="viewport"
		content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no"
	/>
</svelte:head>

<div class="ar-container">
	<a-scene embedded arjs="sourceType: webcam; debugUIEnabled: false;">
		<a-box position="0 0.5 0" material="opacity: 0.5;"></a-box>
		<a-marker-camera preset="hiro"></a-marker-camera>
	</a-scene>
</div>

<style lang="postcss">
	:global(a-scene) {
		position: absolute;
		top: 0;
		left: 0;
		width: 100% !important;
		height: 100% !important;
		overflow: hidden !important;
	}

	.ar-container {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
	}

	:global(.a-canvas) {
		width: 100vw !important;
		height: 100vh !important;
	}

	:global(video) {
		/* position: absolute; */
		/* top: 0; */
		/* left: 0; */
		/* width: 100% !important; */
		/* height: 100% !important; */
		object-fit: cover !important;
		margin: 0 auto !important;
	}

	:global(.a-canvas),
	:global(a-scene) {
		outline: 2px solid red; /* Temporary for debugging */
	}

	:global(video) {
		outline: 2px solid blue; /* Temporary for debugging */
	}
</style>
