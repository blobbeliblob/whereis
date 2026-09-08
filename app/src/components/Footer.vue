<script setup>
  const currentYear = new Date().getFullYear()
</script>

<template>
    <footer class="footer">
        <p>© {{ currentYear }} Camilo Hernández | World Cities Data from SimpleMaps.com <a href="https://creativecommons.org/licenses/by/4.0/" target="_blank" rel="noopener noreferrer">(CC BY 4.0)</a></p>
    </footer>
</template>

<style scoped>
    .footer {
        text-align: center;
        position: fixed;
        bottom: 0;
        width: 100%;
        background: #F5F2E8;
        color: #1A1A18;
        font-size: 0.8rem;
    }
</style>
