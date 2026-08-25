<script setup lang="ts">
defineProps<{
}>()
</script>

<template>
  <div class="loader">
    <div class="loading-wrapper">
      <div></div>
      <div></div>
      <div></div>
      <div></div>
      <div></div>
    </div>
  </div>
</template>

<style scoped>
.loader {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

.loader .loading-wrapper {
  display: flex;
  justify-content: center;
  align-items: flex-end;
  gap: 8px; 
}

.loader .loading-wrapper div {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background-color: var(--defaultDark, #333);
  animation: loading-grow 2s infinite ease-in-out;
}

.loader .loading-wrapper div:nth-child(1) { animation-delay: 0ms; }
.loader .loading-wrapper div:nth-child(2) { animation-delay: 400ms; }
.loader .loading-wrapper div:nth-child(3) { animation-delay: 800ms; }
.loader .loading-wrapper div:nth-child(4) { animation-delay: 1200ms; }
.loader .loading-wrapper div:nth-child(5) { animation-delay: 1600ms; }

@keyframes loading-grow {
  0% {
    transform: translateY(0);
  }
  10% {
    transform: translateY(-20px); 
  }
  20% {
    transform: translateY(0);
  }
  100% {
    transform: translateY(0);
  }
}
</style>