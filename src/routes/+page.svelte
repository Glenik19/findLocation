<script>
  import { onDestroy } from "svelte";
  import { isPointWithinRadius } from "geolib";

  let currentPosition = null;
  let watchId = null;
  let watching = false;

  // Fixes Ziel
  const target = { latitude: 42.05913363079434, longitude: 19.51725368912721 };

  function insideZone() {
    if (!currentPosition) return false;
    return isPointWithinRadius(
      { latitude: currentPosition.coords.latitude, longitude: currentPosition.coords.longitude },
      target,
      5 // Radius in Metern
    );
  }

  function startWatching() {
    if (!("geolocation" in navigator)) return;
    watching = true;
    watchId = navigator.geolocation.watchPosition(
      (pos) => (currentPosition = pos),
      () => (watching = false),
      { enableHighAccuracy: true, maximumAge: 0, timeout: 10000 }
    );
  }

  function stopWatching() {
    if (watchId !== null) {
      navigator.geolocation.clearWatch(watchId);
      watchId = null;
    }
    watching = false;
  }

  onDestroy(() => stopWatching());
</script>

<div class="min-h-screen flex flex-col items-center justify-center bg-gray-50">
  <!-- Start / Stop Button -->
  {#if !watching}
    <button
      class="mb-6 rounded-lg bg-blue-600 px-4 py-2 text-white hover:bg-blue-500"
      on:click={startWatching}
    >
      Start
    </button>
  {:else}
    <button
      class="mb-6 rounded-lg bg-red-600 px-4 py-2 text-white hover:bg-red-500"
      on:click={stopWatching}
    >
      Stop
    </button>
  {/if}

  <!-- Kreis -->
  <div
    class="h-64 w-64 rounded-full border-4 transition-colors duration-500"
    class:bg-green-500={insideZone()}
    class:bg-red-500={!insideZone()}
  ></div>
</div>
