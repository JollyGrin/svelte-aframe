<script lang="ts">
	import { onMount } from 'svelte';

	// State management using runes
	let sceneLoaded = $state(false);
	let hasCamera = $state<boolean | null>(null);
	let showInstructions = $state(true);
	let cameraError = $state<string | null>(null);
	let aframeLoaded = $state(false);
	let debugInfo = $state<string[]>([]);

	function addDebug(info: string) {
	  debugInfo = [...debugInfo, `${new Date().toISOString()}: ${info}`];
	  console.log(info);
	}

	// Load A-Frame dynamically to ensure it's available in the browser
	async function loadAframe() {
	  try {
	    addDebug('Loading A-Frame...');
	    await import('aframe');
	    aframeLoaded = true;
	    addDebug('A-Frame loaded successfully');
	  } catch (error) {
	    addDebug(`Error loading A-Frame: ${error}`);
	    cameraError = "Failed to load components. Please try refreshing the page.";
	  }
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
	            facingMode: 'environment',
	            width: { ideal: 1280 },
	            height: { ideal: 720 }
	          } 
	        });
	        addDebug('Camera permission granted');
	        
	        // Create a video element to test the camera
	        const video = document.createElement('video');
	        video.srcObject = stream;
	        video.style.width = '100%';
	        video.style.height = '100%';
	        video.style.objectFit = 'cover';
	        video.style.position = 'absolute';
	        video.style.top = '0';
	        video.style.left = '0';
	        video.style.zIndex = '0';
	        
	        // Add the video element to test if we can see the camera feed
	        const container = document.querySelector('.camera-test');
	        if (container) {
	          container.appendChild(video);
	          video.play();
	        }
	        
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
	  await loadAframe();
	  if (!aframeLoaded) return;
	  
	  await checkCameraAvailability();
	});

	function closeInstructions() {
	  showInstructions = false;
	}
</script>

<svelte:head>
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" />
</svelte:head>

<!-- Instructions Modal -->
{#if showInstructions}
  <div class="fixed inset-0 z-50 flex items-center justify-center bg-black/80 p-4">
    <div class="w-full max-w-md rounded-lg bg-white p-6">
      <h2 class="mb-4 text-xl font-bold">Camera Test</h2>
      <p class="mb-4">We'll first test if we can access your camera. You should see your camera feed after clicking "Start Test".</p>
      <button
        class="w-full rounded bg-blue-600 px-4 py-2 text-white transition-colors hover:bg-blue-700"
        on:click={closeInstructions}
      >
        Start Test
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
  <div class="fixed bottom-4 left-4 z-50 max-h-48 w-96 overflow-auto rounded bg-black/70 p-4 text-xs text-white">
    <h3 class="mb-2 font-bold">Debug Info:</h3>
    <ul class="space-y-1">
      {#each debugInfo as info}
        <li>{info}</li>
      {/each}
    </ul>
  </div>
{/if}

<!-- Camera Test Container -->
<div class="camera-test relative h-screen w-screen overflow-hidden bg-black">
  {#if !hasCamera}
    <div class="absolute left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 transform text-white">
      No camera available
    </div>
  {/if}
</div>

<style lang="postcss">
  :global(body) {
    margin: 0;
    padding: 0;
    overflow: hidden;
    position: fixed;
    width: 100%;
    height: 100%;
  }
</style>
