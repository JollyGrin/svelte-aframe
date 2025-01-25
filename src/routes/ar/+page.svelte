<script lang="ts">
	import { onMount } from 'svelte';
	import 'aframe';
	import 'aframe-ar';

	// State management using runes
	let sceneLoaded = $state(false);
	let hasCamera = $state<boolean | null>(null);
	let showInstructions = $state(true);
	let cameraError = $state<string | null>(null);

	async function checkCameraAvailability() {
		try {
			const devices = await navigator.mediaDevices.enumerateDevices();
			const videoDevices = devices.filter((device) => device.kind === 'videoinput');
			hasCamera = videoDevices.length > 0;

			if (!hasCamera) {
				cameraError = 'No camera detected on your device. AR requires a camera to work.';
			}
		} catch (error) {
			hasCamera = false;
			cameraError = "Unable to access camera. Please ensure you've granted camera permissions.";
			console.error('Camera check error:', error);
		}
	}

	onMount(() => {
		checkCameraAvailability();

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

	function closeInstructions() {
		showInstructions = false;
	}
</script>

<svelte:head>
	<meta
		name="viewport"
		content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no"
	/>
</svelte:head>

<!-- Instructions Modal -->
{#if showInstructions}
	<div class="fixed inset-0 z-50 flex items-center justify-center bg-black/80 p-4">
		<div class="w-full max-w-md rounded-lg bg-white p-6">
			<h2 class="mb-4 text-xl font-bold">How to Use AR</h2>
			<ol class="mb-6 list-inside list-decimal space-y-2">
				<li>Allow camera access when prompted</li>
				<li>
					Find a Hiro marker (shown below) or <a
						href="https://upload.wikimedia.org/wikipedia/commons/4/48/Hiro_marker.png"
						target="_blank"
						class="text-blue-600 hover:underline">download it here</a
					>
				</li>
				<li>Point your camera at the marker</li>
				<li>A red cube should appear floating above the marker</li>
			</ol>
			<img
				src="https://upload.wikimedia.org/wikipedia/commons/4/48/Hiro_marker.png"
				alt="Hiro Marker"
				class="mx-auto mb-6 h-32 w-32"
			/>
			<button
				class="w-full rounded bg-blue-600 px-4 py-2 text-white transition-colors hover:bg-blue-700"
				on:click={closeInstructions}
			>
				Got it!
			</button>
		</div>
	</div>
{/if}

<!-- Camera Error Modal -->
{#if cameraError}
	<div class="fixed inset-0 z-50 flex items-center justify-center bg-black/80 p-4">
		<div class="w-full max-w-md rounded-lg bg-white p-6">
			<h2 class="mb-4 text-xl font-bold text-red-600">Camera Error</h2>
			<p class="mb-6">{cameraError}</p>
			<div class="space-y-4">
				<p class="text-sm text-gray-600">Troubleshooting steps:</p>
				<ol class="list-inside list-decimal space-y-2 text-sm text-gray-600">
					<li>Make sure your device has a camera</li>
					<li>Allow camera permissions in your browser settings</li>
					<li>Try refreshing the page</li>
					<li>Use a different browser or device</li>
				</ol>
			</div>
		</div>
	</div>
{/if}

<!-- Main AR Scene -->
<div class="ar-container h-screen w-screen">
	{#if hasCamera !== false}
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
		{#if !sceneLoaded && !cameraError}
			<div
				class="absolute left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 transform rounded bg-black/50 p-4 text-white"
			>
				Loading AR Scene...
			</div>
		{/if}
	{/if}
</div>

<style lang="postcss">
	:global(body) {
		margin: 0;
		overflow: hidden;
	}
</style>
