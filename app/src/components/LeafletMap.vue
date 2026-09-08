<script setup>
  import { onBeforeUnmount, onMounted, ref } from 'vue'
  import L from 'leaflet'
  import 'leaflet/dist/leaflet.css'
  import worldCities from '../assets/worldcities.json'

  const mapElement = ref(null);
  const targetLocation = ref('');
  const makeGuessButtonText = ref('Place Guess');
  const populationThresholds = [100000, 250000, 500000, 1000000, 2500000, 5000000, 10000000];
  let populationThresholdIndex = ref(0);
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

    map = L.map(mapElement.value, {
      center: [0, 0], 
      zoom: 3,
      worldCopyJump: true,
      maxBounds: [
        [-90, -Infinity],
        [90, Infinity]
      ], 
      maxBoundsViscosity: 1.0,
    });

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; OpenStreetMap contributors',
      minZoom: 3,
      maxZoom: 10,
    }).addTo(map)

    updatePopulationThreshold();
    
    // game logic

    let currentGuessMarker = null;
    let currentTargetMarker = null;
    let pathBetweenMarkers = null;

    map.on('click', (e) => {
      if (!isGuessing) return;
      if (currentGuessMarker) {
        currentGuessMarker.remove();
      }
      currentGuessMarker = L.marker([e.latlng.lat, e.latlng.lng]).addTo(map);
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
            5000 * (1 - logarithmicPenalty ** 2),
          );
          pointsRound.value = points;
          distanceRound.value = (distance > 1000 ? (distance / 1000).toFixed(distance > 10000 ? 0 : 2).toString() + ' km' : Math.round(distance).toString() + ' m');

          // draw target marker and line between guess and target
          currentTargetMarker = L.marker(targetCoordinates).addTo(map);
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
      <p><span id="pointsRound">{{ pointsRound }}</span> Points!</p>
      <p><span id="distanceRound">{{ distanceRound }}</span> away.</p>
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
    border: 2px solid #1A1A18;
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
    background-color: #1A1A18;
    color: #F5F2E8;
    border: 1px solid #F5F2E8;
    border-radius: 4px;
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
    background-color: #1A1A18;
    color: #F5F2E8;
    border: 1px solid #F5F2E8;
    border-radius: 4px;
  }

  #makeGuessButton {
    display: block;
    padding: 0.5rem 1rem;
    z-index: 1000;
    position: fixed;
    bottom: 10vh;
    font-size: 1rem;
    background-color: #1A1A18;
    color: #F5F2E8;
    border: 1px solid #F5F2E8;
    border-radius: 4px;
    cursor: pointer;
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
    color: #1A1A18;
  }

  #populationThreshold {
    width: 8rem;
    accent-color: #1A1A18;
    cursor: pointer;
  }
</style>
