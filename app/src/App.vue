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
    font-family: 'Satoshi', sans-serif;
    --bauhaus-black: #1A1A18;
    --bauhaus-white: #F5F2E8;
    --bauhaus-red: #C8302A;
    --bauhaus-blue: #1E3878;
    --bauhaus-yellow: #E8C018;
    --bauhaus-gold: #C89620;

    --color-primary: var(--bauhaus-black);
    --color-secondary: var(--bauhaus-white);
    --color-accent: var(--bauhaus-yellow);
    --shadow: 0.2rem -0.1rem rgba(var(--color-primary), 1);
  }

  body {
    min-width: 370px;
    min-height: 400px;
    margin: 0;
    -webkit-tap-highlight-color: transparent;
    color: var(--color-primary);
    background: var(--color-secondary);
  }

  html, body {
    height: 100%;
    width: 100vw;
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
