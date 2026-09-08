<script setup>
  const currentYear = new Date().getFullYear()
</script>

<template>
    <footer class="footer"><p>© {{ currentYear }} cmlodev | World Cities Data from SimpleMaps.com <a href="https://creativecommons.org/licenses/by/4.0/" target="_blank" rel="noopener noreferrer">(CC BY 4.0)</a></p></footer>
</template>

<style scoped>
    .footer {
        text-align: center;
        position: absolute;
        bottom: 0;
        width: 100%;
        font-size: clamp(0.5rem, 2vw, 0.8rem);
    }
</style>
