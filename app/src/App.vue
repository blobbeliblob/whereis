<script setup>
import LeafletMap from './components/LeafletMap.vue'
import Footer from './components/Footer.vue'
</script>

<template>
  <main class="app">
    <h1>Where is?</h1>
    <LeafletMap />
    <Footer />
  </main>
</template>

<style>
:root {
  font-family: Inter, ui-sans-serif, system-ui, sans-serif;
  color: #1A1A18;
  background: #F5F2E8;
}

body {
  margin: 0;
}

.app {
  display: grid;
  gap: 1rem;
  justify-items: start;
  padding: 2rem;
}

h1 {
  margin: 0;
  font-size: clamp(1.5rem, 3vw, 2.25rem);
}

@media (max-width: 35rem) {
  .app {
    padding: 1rem;
  }
}
</style>
