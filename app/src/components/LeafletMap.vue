<script setup>
  import { onBeforeUnmount, onMounted, ref } from 'vue'
  import L from 'leaflet'
  import 'leaflet/dist/leaflet.css'

  const mapElement = ref(null);
  const targetLocation = ref(null);
  let map;

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
      minZoom: 2,
      maxZoom: 10,
    }).addTo(map)
    
    // game logic

    let isGuessing = true;

    let currentGuessMarker = null;
    let currentTargetMarker = null;
    let pathBetweenMarkers = null;

    map.on('click', (e) => {
      if (!isGuessing) return;
      if (currentGuessMarker) {
        currentGuessMarker.remove();
      }
      currentGuessMarker = L.marker([e.latlng.lat, e.latlng.lng]).addTo(map);
    });

    targetLocation.value = "Kölner Dom";

    makeGuessButton.addEventListener('click', () => {
      if (isGuessing) {
        if (currentGuessMarker && targetLocation.value) {
          const guessCoordinates = currentGuessMarker.getLatLng();
          const targetCoordinates = L.latLng(50.941357, 6.958307);
          const distance = map.distance(guessCoordinates, targetCoordinates);
          

          // draw target marker and line between guess and target
          currentTargetMarker = L.marker(targetCoordinates).addTo(map);
          pathBetweenMarkers = L.polyline([guessCoordinates, targetCoordinates], { color: '#C8302A', weight: 2, dashArray: '8, 8' }).addTo(map);
          
          isGuessing = !isGuessing;
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
        isGuessing = !isGuessing;
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
    <div ref="targetLocation" id="targetLocation"></div>
    <div ref="resultsBox" id="resultsBox">
      <div ref="resultsSummary" id="resultsSummary">
        <p><span ref="pointsRound" id="pointsRound"></span> Points!</p>
        <p><span ref="distanceRound" id="distanceRound"></span> meters away.</p>
      </div>
    </div>
    <button ref="makeGuessButton" id="makeGuessButton">Check Guess</button>
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
    height: 87vh;
    width: 100%;
    border: 2px solid #1A1A18;
    border-radius: 8px;
    overflow: hidden;
  }

  #targetLocation {
    display: block;
    padding: 0.5rem 1rem;
    z-index: 1000;
    position: fixed;
    top: 8rem;
    font-size: 1rem;
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
    bottom: 5rem;
    font-size: 1rem;
    background-color: #1A1A18;
    color: #F5F2E8;
    border: 1px solid #F5F2E8;
    border-radius: 4px;
    cursor: pointer;
  }
</style>
