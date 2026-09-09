<script setup>
  import { onBeforeUnmount, onMounted, ref } from 'vue'
  import L from 'leaflet'
  import 'leaflet/dist/leaflet.css'
  import worldCities from '../assets/worldcities.json'

  const mapElement = ref(null);
  const targetLocation = ref('');
  const makeGuessButtonText = ref('Place Guess');
  const maxPointsPerRound = ref(100); // maximum points for a perfect guess
  const populationThresholds = [100000, 250000, 500000, 1000000, 2500000, 5000000, 10000000];
  let populationThresholdIndex = ref(6);
  let pointsRound = ref(0);
  let distanceRound = ref(0);
  let isGuessing = ref(true);
  let targetCity = ref(null);
  let map;;
  let filteredCities = [];

  function filterCitiesByPopulation(populationThreshold) {
    return worldCities.filter(city => city.population > populationThreshold);
  }

  function updatePopulationThreshold() {
    filteredCities = filterCitiesByPopulation(populationThresholds[populationThresholdIndex.value]);
  }

  function getRandomCity() {
    return filteredCities[Math.floor(Math.random() * filteredCities.length)];
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
      "Stadia Toner": Stadia_StamenTonerBackground,
      "Stadia Watercolor": Stadia_StamenWatercolor,
    };

    var layerControl = L.control.layers(baseMaps).addTo(map);

    updatePopulationThreshold();

    const markerIconGuess = L.icon({
      iconUrl: 'https://unpkg.com/leaflet@1.9.4/dist/images/marker-icon.png',
      iconSize: [25, 41],
      iconAnchor: [12, 41],
      popupAnchor: [1, -34],
      shadowUrl: 'https://unpkg.com/leaflet@1.9.4/dist/images/marker-shadow.png',
      shadowSize: [41, 41]
    });

    const markerIconTarget = L.icon({
      iconUrl: 'https://unpkg.com/leaflet@1.9.4/dist/images/marker-icon.png',
      iconSize: [25, 41],
      iconAnchor: [12, 41],
      popupAnchor: [1, -34],
      shadowUrl: 'https://unpkg.com/leaflet@1.9.4/dist/images/marker-shadow.png',
      shadowSize: [41, 41]
    });
    
    // game logic

    let currentGuessMarker = null;
    let currentTargetMarker = null;
    let pathBetweenMarkers = null;

    map.on('click', (e) => {
      if (!isGuessing) return;
      if (currentGuessMarker) {
        currentGuessMarker.remove();
      }
      currentGuessMarker = L.marker([e.latlng.lat, e.latlng.lng], { icon: markerIconGuess }).addTo(map);
      makeGuessButtonText.value = 'Check Guess';
    });

    targetCity.value = getRandomCity();
    targetLocation.value = `${targetCity.value.city}`;

    makeGuessButton.addEventListener('click', () => {
      if (isGuessing) {
        if (currentGuessMarker && targetLocation.value) {
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
          pathBetweenMarkers = L.polyline([guessCoordinates, targetCoordinates], { color: '#C8302A', weight: 2, dashArray: '8, 8' }).addTo(map);
          
          isGuessing = !isGuessing;
          makeGuessButtonText.value = 'Next Round';
        } else {
          // do something
        }
      } else {
        if (currentGuessMarker) {
          currentGuessMarker.remove();
          currentGuessMarker = null;
        }
        if (currentTargetMarker) {
          currentTargetMarker.remove();
          currentTargetMarker = null;
        }
        if (pathBetweenMarkers) {
          pathBetweenMarkers.remove();
          pathBetweenMarkers = null;
        }
        targetCity.value = getRandomCity();
        targetLocation.value = `${targetCity.value.city}`;
        isGuessing = !isGuessing;
        makeGuessButtonText.value = 'Place Guess';
      }
    });

  })

  onBeforeUnmount(() => {
    map?.remove()
  })
</script>

<template>
  <div id="mainContent">
    <div ref="mapElement" class="map" aria-label="Interactive Map"></div>
    <div id="targetLocation">{{ targetLocation }}</div>
    <div id="resultsBox" v-show="!isGuessing">
      <p>You got <span id="pointsRound">{{ pointsRound }}</span> / <span id="maxPointsPerRound">{{ maxPointsPerRound }}</span> points,</p>
      <p>and were <span id="distanceRound">{{ distanceRound }}</span> away!</p>
    </div>
    <button ref="makeGuessButton" id="makeGuessButton">{{ makeGuessButtonText }}</button>
    <label id="populationThresholdLabel" for="populationThreshold">
      > {{ populationThresholds[populationThresholdIndex].toLocaleString() }} pop.
    </label>
    <input
      id="populationThreshold"
      v-model.number="populationThresholdIndex"
      type="range"
      min="0"
      max="6"
      step="1"
      list="populationThresholdValues"
      aria-label="Minimum city population"
      @input="updatePopulationThreshold"
    />
    <datalist id="populationThresholdValues">
      <option v-for="(threshold, index) in populationThresholds" :key="threshold" :value="index">
        {{ threshold.toLocaleString() }}
      </option>
    </datalist>
  
  </div>
</template>

<style scoped>
  #mainContent {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 100%;
  }

  .map {
    height: 85vh;
    width: 100%;
    border: 2px solid var(--color-primary);
    border-radius: 8px;
    overflow: hidden;
    cursor: crosshair;
  }

  #targetLocation {
    display: block;
    padding: 0.5rem 1rem;
    z-index: 1000;
    position: fixed;
    top: 5rem;
    font-size: 1.4rem;
    text-transform: uppercase;
    font-weight: bold;
    background-color: var(--color-secondary);
    color: var(--color-primary);
    border: 2px solid var(--color-primary);
    border-radius: 4px;
    box-shadow: var(--shadow);
  }

  #resultsBox {
    display: block;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    padding: 0.5rem 1rem;
    z-index: 1000;
    font-size: 1rem;
    background-color: var(--color-secondary);
    color: var(--color-primary);
    border: 2px solid var(--color-primary);
    border-radius: 4px;
    box-shadow: var(--shadow);
  }

  #pointsRound {
    font-size: 1.2rem;
    font-weight: bold;
    color: var(--bauhaus-red);
  }

  #distanceRound {
    font-weight: bold;
    color: var(--bauhaus-blue);
  }

  #makeGuessButton {
    display: block;
    padding: 0.5rem 1rem;
    z-index: 1000;
    position: fixed;
    bottom: 10vh;
    font-size: 1rem;
    background-color: var(--color-accent);
    color: var(--color-primary);
    border: 2px solid var(--color-primary);
    border-radius: 4px;
    cursor: pointer;
  }

  #makeGuessButton:hover {
    background-color: var(--color-primary);
    color: var(--color-accent);
  }

  #populationThresholdLabel,
  #populationThreshold {
    position: fixed;
    top: 2.2rem;
    right: 2rem;
    z-index: 1000;
  }

  #populationThresholdLabel {
    top: 1.5rem;
    right: 2.6rem;
    font-size: 0.8rem;
    color: var(--color-primary);
  }

  #populationThreshold {
    width: 8rem;
    accent-color: var(--color-primary);
    cursor: pointer;
  }
</style>
