<script setup>
  import { computed, onBeforeUnmount, onMounted, ref } from 'vue'
  import L from 'leaflet'
  import 'leaflet/dist/leaflet.css'
  import worldCities from '../assets/worldcities.json'

  const mapElement = ref(null);
  const targetLocation = ref('');
  const maxPointsPerRound = ref(100); // maximum points for a perfect guess
  const populationThresholds = [100000, 250000, 500000, 1000000, 2500000, 5000000, 10000000];
  const populationThresholdIndex = ref(6);
  const pointsRound = ref(0);
  const distanceRound = ref(0);
  const isGuessing = ref(true);
  const hasGuess = ref(false);
  const targetCity = ref(null);
  let map;
  let filteredCities = [];

  let currentGuessMarker = null;
  let currentTargetMarker = null;
  let pathBetweenMarkers = null;

  const makeGuessButtonText = computed(() => {
    if (!isGuessing.value) return 'Next round';
    return hasGuess.value ? 'Check guess' : 'Place guess';
  });

  const populationLabel = computed(() =>
    new Intl.NumberFormat('en', { notation: 'compact' }).format(populationThresholds[populationThresholdIndex.value])
  );

  function filterCitiesByPopulation(populationThreshold) {
    return worldCities.filter(city => city.population > populationThreshold);
  }

  function updatePopulationThreshold() {
    filteredCities = filterCitiesByPopulation(populationThresholds[populationThresholdIndex.value]);
  }

  function getRandomCity() {
    return filteredCities[Math.floor(Math.random() * filteredCities.length)];
  }

  // Swiss-style markers: flat dots instead of the default blue pin
  const markerIconGuess = L.divIcon({ className: 'marker marker--guess', iconSize: [18, 18], iconAnchor: [9, 9] });
  const markerIconTarget = L.divIcon({ className: 'marker marker--target', iconSize: [18, 18], iconAnchor: [9, 9] });

  function onActionClick() {
    if (isGuessing.value) {
      if (!currentGuessMarker || !targetLocation.value) return;

      const guessCoordinates = currentGuessMarker.getLatLng();
      const targetCoordinates = L.latLng(targetCity.value.lat, targetCity.value.lng);
      const distance = map.distance(guessCoordinates, targetCoordinates);

      const thresholdDistance = 2500000; // in meters
      const allowedError = 5000; // in meters
      const scoringDistance = Math.min(thresholdDistance, Math.max(allowedError, distance));
      const logarithmicPenalty =
        Math.log10(scoringDistance / allowedError) / Math.log10(thresholdDistance / allowedError);
      const points = Math.round(
        maxPointsPerRound.value * (1 - logarithmicPenalty ** 2),
      );
      pointsRound.value = points;
      distanceRound.value = (distance > 1000 ? (distance / 1000).toFixed(distance > 10000 ? 0 : 2).toString() + ' km' : Math.round(distance).toString() + ' m');

      // draw target marker and line between guess and target
      currentTargetMarker = L.marker(targetCoordinates, { icon: markerIconTarget }).addTo(map);
      pathBetweenMarkers = L.polyline([guessCoordinates, targetCoordinates], { color: '#121417', weight: 2, dashArray: '2, 8', lineCap: 'round' }).addTo(map);
      map.flyToBounds(L.latLngBounds([guessCoordinates, targetCoordinates]), { padding: [80, 80], maxZoom: 8, duration: 0.8 });

      isGuessing.value = false;
    } else {
      currentGuessMarker?.remove();
      currentTargetMarker?.remove();
      pathBetweenMarkers?.remove();
      currentGuessMarker = currentTargetMarker = pathBetweenMarkers = null;
      hasGuess.value = false;

      targetCity.value = getRandomCity();
      targetLocation.value = `${targetCity.value.city}`;
      isGuessing.value = true;
    }
  }

  onMounted(() => {
    // map setup

    // more basemaps can be found at https://leaflet-extras.github.io/leaflet-providers/preview/
    var Stadia_StamenTonerBackground = L.tileLayer('https://tiles.stadiamaps.com/tiles/stamen_toner_background/{z}/{x}/{y}{r}.{ext}', {
      minZoom: 3,
      maxZoom: 12,
      attribution: '&copy; <a href="https://www.stadiamaps.com/" target="_blank">Stadia Maps</a> &copy; <a href="https://www.stamen.com/" target="_blank">Stamen Design</a> &copy; <a href="https://openmaptiles.org/" target="_blank">OpenMapTiles</a> &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      ext: 'png'
    });
    var Stadia_StamenWatercolor = L.tileLayer('https://tiles.stadiamaps.com/tiles/stamen_watercolor/{z}/{x}/{y}.{ext}', {
      minZoom: 3,
      maxZoom: 12,
      attribution: '&copy; <a href="https://www.stadiamaps.com/" target="_blank">Stadia Maps</a> &copy; <a href="https://www.stamen.com/" target="_blank">Stamen Design</a> &copy; <a href="https://openmaptiles.org/" target="_blank">OpenMapTiles</a> &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      ext: 'jpg'
    });

    map = L.map(mapElement.value, {
      center: [0, 0],
      zoom: 3,
      worldCopyJump: true,
      maxBounds: [
        [-90, -Infinity],
        [90, Infinity]
      ],
      maxBoundsViscosity: 1.0,
      layers: [Stadia_StamenWatercolor]
    });

    var baseMaps = {
      "Toner": Stadia_StamenTonerBackground,
      "Watercolor": Stadia_StamenWatercolor,
    };

    L.control.layers(baseMaps).addTo(map);

    updatePopulationThreshold();

    // game logic

    map.on('click', (e) => {
      if (!isGuessing.value) return;
      currentGuessMarker?.remove();
      currentGuessMarker = L.marker([e.latlng.lat, e.latlng.lng], { icon: markerIconGuess }).addTo(map);
      hasGuess.value = true;
    });

    targetCity.value = getRandomCity();
    targetLocation.value = `${targetCity.value.city}`;
  })

  onBeforeUnmount(() => {
    map?.remove()
  })
</script>

<template>
  <div class="game">
    <header class="brand">
      <slot name="title" />
    </header>

    <section class="block target" aria-live="polite">
      <span class="label"><span class="num">01</span> Find</span>
      <p class="city">{{ targetLocation }}</p>
    </section>

    <section class="block result" aria-live="polite">
      <span class="label"><span class="num">02</span> {{ isGuessing ? 'Guess' : 'Result' }}</span>
      <template v-if="isGuessing">
        <p class="hint">{{ hasGuess ? 'Happy with it? Check your guess.' : 'Tap the map where you think it is.' }}</p>
      </template>
      <template v-else>
        <p class="score">
          <span class="points">{{ pointsRound }}</span><span class="max">/{{ maxPointsPerRound }}</span>
        </p>
        <p class="distance">{{ distanceRound }} away</p>
      </template>
    </section>

    <button
      class="action"
      :class="{ 'is-ready': hasGuess || !isGuessing }"
      :disabled="isGuessing && !hasGuess"
      @click="onActionClick"
    >
      <span>{{ makeGuessButtonText }}</span>
      <span class="arrow" aria-hidden="true">→</span>
    </button>

    <section class="block filter">
      <label class="label" for="populationThreshold"><span class="num">03</span> Min. pop.</label>
      <div class="filter-row">
        <span class="filter-value">{{ populationLabel }}</span>
        <input
          id="populationThreshold"
          v-model.number="populationThresholdIndex"
          type="range"
          min="0"
          max="6"
          step="1"
          aria-label="Minimum city population"
          :aria-valuetext="populationThresholds[populationThresholdIndex].toLocaleString()"
          :style="{ '--fill': (populationThresholdIndex / 6) * 100 + '%' }"
          @input="updatePopulationThreshold"
        />
      </div>
    </section>

    <div class="footer-slot">
      <slot name="footer" />
    </div>

    <div class="map-well">
      <div ref="mapElement" class="map" aria-label="Interactive Map"></div>
    </div>
  </div>
</template>

<style scoped>
  /* ---------- Grid ---------- */

  .game {
    height: 100%;
    display: grid;
    gap: var(--gap);
    grid-template-columns: minmax(260px, 340px) 1fr;
    grid-template-rows: auto auto auto auto 1fr auto;
    grid-template-areas:
      'brand  map'
      'target map'
      'result map'
      'action map'
      'filter map'
      'footer map';
  }

  .brand       { grid-area: brand; }
  .target      { grid-area: target; }
  .result      { grid-area: result; }
  .action      { grid-area: action; }
  .filter      { grid-area: filter; align-self: end; }
  .footer-slot { grid-area: footer; }
  .map-well    { grid-area: map; }

  .brand {
    padding: 0.25rem 0.25rem 0.5rem;
    border-bottom: 2px solid var(--ink);
  }

  /* ---------- Neumorphic blocks ---------- */

  .block {
    display: flex;
    flex-direction: column;
    gap: 0.6rem;
    padding: 1.25rem 1.4rem;
    border-radius: var(--radius-md);
    background: var(--surface);
    box-shadow: var(--raise);
    min-width: 0;
  }

  .label {
    display: flex;
    align-items: baseline;
    gap: 0.6rem;
    font-size: 0.7rem;
    font-weight: 600;
    letter-spacing: 0.12em;
    text-transform: uppercase;
    color: var(--ink-muted);
  }

  .num {
    color: var(--accent);
    font-weight: 700;
  }

  .city {
    margin: 0;
    font-size: clamp(1.75rem, 3.2vw, 2.75rem);
    font-weight: 800;
    line-height: 1;
    letter-spacing: -0.035em;
    overflow-wrap: anywhere;
  }

  .result {
    min-height: 8.5rem;
    justify-content: flex-start;
  }

  .hint {
    margin: 0;
    font-size: 0.95rem;
    line-height: 1.4;
    color: var(--ink-muted);
  }

  .score {
    margin: 0;
    display: flex;
    align-items: baseline;
    line-height: 0.9;
  }

  .points {
    font-size: clamp(2.75rem, 5vw, 4rem);
    font-weight: 800;
    letter-spacing: -0.05em;
    color: var(--accent);
  }

  .max {
    margin-left: 0.2rem;
    font-size: 1.1rem;
    font-weight: 600;
    color: var(--ink-muted);
  }

  .distance {
    margin: 0;
    font-size: 0.95rem;
    font-weight: 600;
  }

  /* ---------- Primary action ---------- */

  .action {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    padding: 1.1rem 1.4rem;
    border: none;
    border-radius: var(--radius-md);
    background: var(--surface);
    box-shadow: var(--raise);
    font: inherit;
    font-size: 1rem;
    font-weight: 700;
    letter-spacing: -0.01em;
    color: var(--ink);
    cursor: pointer;
    transition: box-shadow 160ms ease, color 160ms ease, transform 160ms ease;
  }

  .action .arrow {
    display: grid;
    place-items: center;
    width: 2.25rem;
    height: 2.25rem;
    border-radius: 50%;
    box-shadow: var(--inset-sm);
    font-size: 1.1rem;
    transition: background 160ms ease, color 160ms ease, box-shadow 160ms ease;
  }

  .action.is-ready .arrow {
    background: var(--accent);
    color: #fff;
    box-shadow: 3px 3px 8px rgba(226, 35, 26, 0.35), -3px -3px 8px rgba(255, 255, 255, 0.9);
  }

  .action:hover:not(:disabled) {
    box-shadow: var(--raise-sm);
  }

  .action:active:not(:disabled) {
    box-shadow: var(--inset);
    transform: translateY(1px);
  }

  .action:disabled {
    cursor: default;
    color: var(--ink-muted);
  }

  .action:focus-visible,
  input[type='range']:focus-visible {
    outline: 2px solid var(--ink);
    outline-offset: 4px;
  }

  /* ---------- Range slider ---------- */

  .filter-row {
    display: flex;
    align-items: center;
    gap: 1rem;
  }

  .filter-value {
    min-width: 3.2ch;
    font-size: 1.5rem;
    font-weight: 800;
    letter-spacing: -0.03em;
  }

  input[type='range'] {
    flex: 1;
    min-width: 0;
    height: 12px;
    margin: 0;
    border-radius: 999px;
    background:
      linear-gradient(var(--ink), var(--ink)) 0 50% / var(--fill, 100%) 2px no-repeat,
      var(--surface);
    box-shadow: var(--inset-sm);
    appearance: none;
    -webkit-appearance: none;
    cursor: pointer;
  }

  input[type='range']::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 26px;
    height: 26px;
    border-radius: 50%;
    border: none;
    background: radial-gradient(circle, var(--accent) 0 3px, var(--surface) 3.5px);
    box-shadow: var(--raise-sm);
  }

  input[type='range']::-moz-range-thumb {
    width: 26px;
    height: 26px;
    border-radius: 50%;
    border: none;
    background: radial-gradient(circle, var(--accent) 0 3px, var(--surface) 3.5px);
    box-shadow: var(--raise-sm);
  }

  /* ---------- Map well ---------- */

  .map-well {
    min-height: 0;
    padding: 10px;
    border-radius: var(--radius-lg);
    background: var(--surface);
    box-shadow: var(--inset);
  }

  .map {
    height: 100%;
    width: 100%;
    border-radius: calc(var(--radius-lg) - 10px);
    overflow: hidden;
    cursor: crosshair;
    background: var(--surface);
  }

  /* Leaflet controls, re-skinned to match the surface */
  .map :deep(.leaflet-control-zoom),
  .map :deep(.leaflet-control-layers) {
    border: none;
    border-radius: var(--radius-sm);
    background: var(--surface);
    box-shadow: 4px 4px 12px rgba(18, 20, 23, 0.18);
    overflow: hidden;
  }

  .map :deep(.leaflet-control-zoom a) {
    width: 36px;
    height: 36px;
    line-height: 36px;
    border: none;
    background: var(--surface);
    color: var(--ink);
    font-weight: 400;
  }

  .map :deep(.leaflet-control-zoom a:first-child) {
    border-bottom: 1px solid rgba(18, 20, 23, 0.1);
  }

  .map :deep(.leaflet-control-zoom a:hover) {
    background: var(--surface);
    box-shadow: var(--inset-sm);
  }

  .map :deep(.leaflet-control-layers-expanded) {
    padding: 0.75rem 1rem;
    font: inherit;
    font-size: 0.8rem;
    color: var(--ink);
  }

  .map :deep(.leaflet-control-layers-selector) {
    accent-color: var(--accent);
  }

  .map :deep(.leaflet-control-attribution) {
    border-top-left-radius: 8px;
    background: rgba(228, 230, 234, 0.85);
    font-size: 0.6rem;
    color: var(--ink-muted);
  }

  /* Markers */
  .map :deep(.marker) {
    border-radius: 50%;
    border: 3px solid #fff;
    box-shadow: 0 2px 8px rgba(18, 20, 23, 0.35);
  }

  .map :deep(.marker--guess) {
    background: var(--ink);
  }

  .map :deep(.marker--target) {
    background: var(--accent);
    animation: pulse 1.6s ease-out infinite;
  }

  @keyframes pulse {
    0%   { box-shadow: 0 0 0 0 rgba(226, 35, 26, 0.45), 0 2px 8px rgba(18, 20, 23, 0.35); }
    100% { box-shadow: 0 0 0 16px rgba(226, 35, 26, 0), 0 2px 8px rgba(18, 20, 23, 0.35); }
  }

  /* ---------- Compact layout ---------- */

  @media (max-width: 860px) {
    .game {
      gap: 0.9rem;
      grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
      grid-template-rows: auto auto minmax(0, 1fr) auto auto;
      grid-template-areas:
        'brand  filter'
        'target target'
        'map    map'
        'result action'
        'footer footer';
    }

    h1, :slotted(h1) {
      font-size: 1.9rem;
    }

    .brand {
      align-self: end;
      border-bottom: none;
      padding: 0;
    }

    .block {
      padding: 0.9rem 1rem;
      gap: 0.4rem;
    }

    .target {
      flex-direction: row;
      align-items: baseline;
      gap: 1rem;
    }

    .city {
      font-size: 1.75rem;
    }

    .filter {
      align-self: stretch;
      box-shadow: var(--raise-sm);
    }

    .filter-value {
      font-size: 1.1rem;
    }

    .filter .label {
      white-space: nowrap;
      letter-spacing: 0.08em;
    }

    .filter-row {
      gap: 0.75rem;
    }

    .result {
      min-height: 0;
    }

    .hint {
      font-size: 0.8rem;
    }

    .points {
      font-size: 2.25rem;
    }

    .action {
      align-self: stretch;
      padding: 0.9rem 1rem;
    }

    .map-well {
      padding: 7px;
      border-radius: var(--radius-md);
    }

    .map {
      border-radius: calc(var(--radius-md) - 7px);
    }
  }
</style>
