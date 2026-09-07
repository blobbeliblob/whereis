<script setup>
  import { onBeforeUnmount, onMounted, ref } from 'vue'
  import L from 'leaflet'
  import 'leaflet/dist/leaflet.css'

  const mapElement = ref(null);
  let map;

  function getLocationCoordinates() {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        (position) => {
          return [position.coords.latitude, position.coords.longitude];
        },
        (error) => {
          console.error('Error getting location:', error);
        }
      )
    } else {
      console.error('Geolocation is not supported by this browser.');
    }
    return [50.941357, 6.958307];  // default to Kölner Dom if no location is available
  }

  // create a marker at the given location
  function createMarker(coordinates) {
    const formattedCoordinates = coordinates.map(coord => coord.toFixed(4)).join(', ');
    L.marker(coordinates)
      .addTo(map)
      .bindPopup(formattedCoordinates)
      .openPopup();
  }

  onMounted(() => {
    const locationCoordinates = getLocationCoordinates()

    map = L.map(mapElement.value).setView(locationCoordinates, 10)

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; OpenStreetMap contributors',
      minZoom: 2,
      maxZoom: 10,
    }).addTo(map)
    
    let currentGuessMarker = null;

    map.on('click', (e) => {
      if (currentGuessMarker) {
        currentGuessMarker.remove();
      }
      currentGuessMarker = L.marker([e.latlng.lat, e.latlng.lng]).addTo(map);
    });
  })

  onBeforeUnmount(() => {
    map?.remove()
  })
</script>

<template>
  <div ref="mapElement" class="map" aria-label="Interactive Map"></div>

</template>

<style scoped>
  .map {
    height: 87vh;
    width: 100%;
    border: 2px solid #1A1A18;
    overflow: hidden;
  }
</style>
