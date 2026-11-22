<script setup>
  const currentYear = new Date().getFullYear()
</script>

<template>
  <footer class="footer">
    <p>© {{ currentYear }} cmlodev</p>
    <p>
      World cities data — SimpleMaps.com
      <a href="https://creativecommons.org/licenses/by/4.0/" target="_blank" rel="noopener noreferrer">CC BY 4.0</a>
    </p>
  </footer>
</template>

<style scoped>
  .footer {
    font-size: 0.68rem;
    line-height: 1.5;
    letter-spacing: 0.02em;
    color: var(--ink-muted);
  }

  .footer p {
    margin: 0;
  }

  .footer a {
    border-bottom: 1px solid currentColor;
  }

  .footer a:hover {
    color: var(--accent);
  }
</style>
