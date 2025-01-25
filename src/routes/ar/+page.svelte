<script lang="ts">
	import { onMount } from 'svelte';

	// State management using runes
	let sceneLoaded = $state(false);
	let hasCamera = $state<boolean | null>(null);
	let showInstructions = $state(true);
	let cameraError = $state<string | null>(null);
	let debugInfo = $state<string[]>([]);

	function addDebug(info: string) {
		debugInfo = [...debugInfo, `${new Date().toISOString()}: ${info}`];
		console.log(info);
	}

	async function checkCameraAvailability() {
		try {
			addDebug('Checking camera devices...');
			const devices = await navigator.mediaDevices.enumerateDevices();
			const videoDevices = devices.filter((device) => device.kind === 'videoinput');
			hasCamera = videoDevices.length > 0;
			addDebug(`Found ${videoDevices.length} video devices`);

			if (!hasCamera) {
				cameraError = 'No camera detected on your device. AR requires a camera to work.';
			} else {
				try {
					addDebug('Requesting camera permission...');
					const stream = await navigator.mediaDevices.getUserMedia({
						video: {
							facingMode: 'environment'
						}
					});
					addDebug('Camera permission granted');
					stream.getTracks().forEach((track) => track.stop());
					addDebug('Camera ready for AR');
				} catch (error) {
					addDebug(`Camera permission error: ${error}`);
					cameraError = "Unable to access camera. Please ensure you've granted camera permissions.";
				}
			}
		} catch (error) {
			hasCamera = false;
			addDebug(`Camera check error: ${error}`);
			cameraError = "Unable to check camera availability. Please ensure you're using a modern browser.";
		}
	}

	onMount(async () => {
		await checkCameraAvailability();

		// Initialize AR scene
		const scene = document.querySelector('a-scene');
		if (scene?.hasLoaded) {
			addDebug('Scene already loaded');
			sceneLoaded = true;
		} else {
			addDebug('Waiting for scene to load...');
			scene?.addEventListener('loaded', () => {
				addDebug('Scene loaded');
				sceneLoaded = true;
			});
			scene?.addEventListener('camera-init', () => {
				addDebug('Camera initialized in AR');
			});
			scene?.addEventListener('camera-error', (error) => {
				addDebug(`AR Camera error: ${error}`);
			});
		}
	});

	function closeInstructions() {
		showInstructions = false;
	}
</script>

<svelte:head>
	<script src="https://aframe.io/releases/1.4.2/aframe.min.js"></script>
	<script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar.js"></script>
	<meta
		name="viewport"
		content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no"
	/>
</svelte:head>

<!-- Instructions Modal -->
{#if showInstructions}
	<div class="fixed inset-0 z-50 flex items-center justify-center bg-black/80 p-4">
		<div class="w-full max-w-md rounded-lg bg-white p-6">
			<h2 class="mb-4 text-xl font-bold">AR Camera Test</h2>
			<p class="mb-4">
				We'll activate your camera and look for the Hiro marker. You should see your camera feed
				after clicking "Start".
			</p>
			<img
				src="https://raw.githubusercontent.com/AR-js-org/AR.js/master/data/images/hiro.png"
				alt="Hiro Marker"
				class="mx-auto mb-6 h-32 w-32"
			/>
			<button
				class="w-full rounded bg-blue-600 px-4 py-2 text-white transition-colors hover:bg-blue-700"
				on:click={closeInstructions}
			>
				Start AR
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

<!-- Debug Info -->
{#if import.meta.env.DEV}
	<div
		class="fixed bottom-4 left-4 z-50 max-h-48 w-96 overflow-auto rounded bg-black/70 p-4 text-xs text-white"
	>
		<h3 class="mb-2 font-bold">Debug Info:</h3>
		<ul class="space-y-1">
			{#each debugInfo as info}
				<li>{info}</li>
			{/each}
		</ul>
	</div>
{/if}

<!-- AR Scene -->
{#if hasCamera !== false}
	<div class="ar-scene">
		<a-scene
			embedded
			vr-mode-ui="enabled: false"
			loading-screen="enabled: false"
			renderer="antialias: true;"
			arjs="sourceType: webcam; videoTexture: true; debugUIEnabled: true; detectionMode: mono; trackingMethod: best; areaThreshold: 0.1; patternRatio: 0.75;"
		>
			<a-marker 
				preset="hiro"
				emitEvents="true"
				on:markerFound={() => addDebug('Marker found!')}
				on:markerLost={() => addDebug('Marker lost!')}
			>
				<!-- Add a simple red box and a text label for visibility -->
				<a-box position="0 0.5 0" material="color: red;" scale="1 1 1"></a-box>
				<a-text value="Marker Detected!" position="0 1 0" align="center" color="white" scale="2 2 2"></a-text>
			</a-marker>

			<a-entity camera></a-entity>
		</a-scene>
	</div>
{/if}

<style lang="postcss">
	:global(body) {
		margin: 0;
		padding: 0;
		overflow: hidden;
		position: fixed;
		width: 100%;
		height: 100%;
		background-color: transparent;
	}

	.ar-scene {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
	}

	:global(.a-canvas) {
		width: 100% !important;
		height: 100% !important;
	}

	:global(video) {
		object-fit: cover !important;
	}
</style>
