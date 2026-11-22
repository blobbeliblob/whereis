<script setup>
  import LeafletMap from './components/LeafletMap.vue'
  import Footer from './components/Footer.vue'
</script>

<template>
  <main class="app">
    <LeafletMap>
      <template #title>
        <h1>Where<br />is?</h1>
      </template>
      <template #footer>
        <Footer />
      </template>
    </LeafletMap>
  </main>
</template>

<style>
  :root {
    /* Swiss type: one neo-grotesk family, hierarchy through size and weight */
    font-family: 'Inter', 'Helvetica Neue', Helvetica, Arial, sans-serif;
    font-feature-settings: 'ss01', 'cv11', 'tnum';
    -webkit-font-smoothing: antialiased;

    /* Neumorphic surface: everything sits on a single soft material */
    --surface: #e4e6ea;
    --surface-light: #ffffff;
    --surface-dark: #a7adb8;
    --ink: #121417;
    --ink-muted: #6b717c;
    --accent: #e2231a; /* the single Swiss red */

    --radius-lg: 28px;
    --radius-md: 18px;
    --radius-sm: 12px;

    --raise: 10px 10px 22px rgba(167, 173, 184, 0.65), -10px -10px 22px rgba(255, 255, 255, 0.9);
    --raise-sm: 5px 5px 10px rgba(167, 173, 184, 0.6), -5px -5px 10px rgba(255, 255, 255, 0.9);
    --inset: inset 7px 7px 14px rgba(167, 173, 184, 0.6), inset -7px -7px 14px rgba(255, 255, 255, 0.85);
    --inset-sm: inset 3px 3px 6px rgba(167, 173, 184, 0.6), inset -3px -3px 6px rgba(255, 255, 255, 0.85);

    --gap: clamp(1rem, 2.2vw, 1.75rem);
  }

  *, *::before, *::after {
    box-sizing: border-box;
  }

  html, body {
    height: 100%;
    margin: 0;
  }

  body {
    min-width: 360px;
    min-height: 480px;
    -webkit-tap-highlight-color: transparent;
    color: var(--ink);
    background: var(--surface);
  }

  .app {
    height: 100dvh;
    padding: var(--gap);
  }

  h1 {
    margin: 0;
    font-size: clamp(2rem, 4.2vw, 3.5rem);
    font-weight: 800;
    line-height: 0.92;
    letter-spacing: -0.045em;
  }

  a {
    color: inherit;
    text-decoration: none;
  }
</style>
