<script>
  import { onDestroy } from "svelte";
  import { getDistance, getGreatCircleBearing } from "geolib";

  let currentPosition = null;
  let watchId = null;
  let watching = false;

  const target = { latitude: 42.05913363079434, longitude: 19.51725368912721 };

  function distanceToTarget() {
    if (!currentPosition) return null;
    return getDistance(
      { latitude: currentPosition.coords.latitude, longitude: currentPosition.coords.longitude },
      target
    );
  }

  function insideZone() {
    const dist = distanceToTarget();
    return dist !== null && dist <= 5;
  }

  function bearingToTarget() {
    if (!currentPosition) return 0;
    return getGreatCircleBearing(
      { latitude: currentPosition.coords.latitude, longitude: currentPosition.coords.longitude },
      target
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

<!-- Fixed Container -->
<div class="fixed top-0 left-0 w-full h-[100dvh] flex flex-col items-center justify-center bg-gray-900">

  <!-- Start / Stop Buttons -->
  <div class="mb-6 flex gap-4">
    {#if !watching}
      <button class="rounded-lg bg-blue-600 px-6 py-3 text-white font-semibold shadow hover:bg-blue-500 transition" on:click={startWatching}>Start</button>
    {:else}
      <button class="rounded-lg bg-red-600 px-6 py-3 text-white font-semibold shadow hover:bg-red-500 transition" on:click={stopWatching}>Stop</button>
    {/if}
  </div>

  <!-- Kreis mit Pfeil -->
  <div class="relative flex items-center justify-center h-80 w-80 rounded-full border-8 border-gray-700 transition-colors duration-500"
       class:bg-green-500={insideZone()}
       class:bg-red-600={!insideZone()}>

    {#if currentPosition}
      <!-- Pfeil nach außen -->
      <div class="absolute text-white text-5xl" style="transform: rotate({bearingToTarget()}deg) translateY(-120px);">
        ▲
      </div>

      <!-- Distanzanzeige -->
      <div class="absolute bottom-6 text-2xl font-bold text-white drop-shadow">
        {distanceToTarget()} m
      </div>
    {/if}
  </div>
</div>
