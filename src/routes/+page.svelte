<script>
  import { onDestroy } from "svelte";
  import { getDistance, getGreatCircleBearing } from "geolib";

  let currentPosition = null;
  let watchId = null;
  let watching = false;

  const target = { latitude: 42.05913363079434, longitude: 19.51725368912721 };

  // Sensorwerte
  let alpha = 0;
  let beta = 0;
  let gamma = 0;

  // Berechnungen
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

  // Fixierter Bearing vom User zum Ziel
  function bearingToTarget() {
    if (!currentPosition) return 0;
    return getGreatCircleBearing(
      { latitude: currentPosition.coords.latitude, longitude: currentPosition.coords.longitude },
      target
    );
  }

  // GPS starten / stoppen
  function startWatching() {
    if (!("geolocation" in navigator)) return;
    watching = true;
    watchId = navigator.geolocation.watchPosition(
      (pos) => (currentPosition = pos),
      () => (watching = false),
      { enableHighAccuracy: true, maximumAge: 0, timeout: 10000 }
    );

    // Orientation Events starten
    window.addEventListener("deviceorientationabsolute", handleOrientation, true);
  }

  function stopWatching() {
    if (watchId !== null) {
      navigator.geolocation.clearWatch(watchId);
      watchId = null;
    }
    watching = false;
    window.removeEventListener("deviceorientationabsolute", handleOrientation, true);
  }

  function handleOrientation(event) {
    alpha = Math.round(event.alpha ?? 0); // Heading (Kompass)
    beta = Math.round(event.beta ?? 0);
    gamma = Math.round(event.gamma ?? 0);
  }

  // Pfeilrichtung = Zielrichtung - Geräteausrichtung
  function arrowRotation() {
    if (!currentPosition) return 0;
    let bearing = bearingToTarget(); // Richtung vom User → Ziel
    let heading = alpha; // Aktuelle Blickrichtung
    return bearing - heading; // Differenz = wo der Pfeil hinzeigen soll
  }

  onDestroy(() => stopWatching());
</script>

<!-- Fixed Fullscreen Container -->
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
      <!-- Weißer Pfeil -->
      <div 
        class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2"
        style="transform: rotate({arrowRotation()}deg) translateY(-120px);"
      >
        <svg xmlns="http://www.w3.org/2000/svg" class="h-14 w-14 text-white drop-shadow" fill="currentColor" viewBox="0 0 24 24">
          <path d="M12 2l6 10H6l6-10z" />
        </svg>
      </div>

      <!-- Distanzanzeige -->
      <div class="absolute bottom-6 text-2xl font-bold text-white drop-shadow">
        {distanceToTarget()} m
      </div>
    {/if}
  </div>

  <!-- Orientation Debug -->
  <div class="mt-8 text-white text-lg font-mono text-center">
    <p>(Alpha): {alpha}°</p>
    <p>(Beta): {beta}°</p>
    <p>(Gamma): {gamma}°</p>
  </div>
</div>
