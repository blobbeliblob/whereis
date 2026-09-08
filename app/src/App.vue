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
    min-width: 370px;
    min-height: 400px;
    margin: 0;
  }

  .app {
    padding: 1.5rem;
  }

  h1 {
    margin: 0 0 1rem 0;
    font-size: 1.5rem;
  }

  a {
    text-decoration: none;
  }
</style>
