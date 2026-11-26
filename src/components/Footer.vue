<script setup lang="ts">
import { RouterLink } from 'vue-router'

defineProps<{
}>()

</script>

<template>

  <footer>
    <div class="footer-wrapper">
      <RouterLink to="/">
        <div class="logo">✒<span>L</span>at</div>
      </RouterLink>

      <p class="signature">Created by <a href="https://github.com/IoanaGeorgia">Ioana Georgia</a></p>

    </div>

  </footer>



</template>

<style scoped>
.footer-wrapper{
  min-height:var(--headerHeight);
  background-color:var(--defaultDark);
  padding:var(--defaultSmallPadding);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  color:#FFF;
  font-size:3rem;
}

.footer-wrapper .signature{
  color:grey;
  font-size:0.7rem;
  margin-top:10px;
  margin-bottom:10px;
}

.footer-wrapper .signature a{
  color:var(--accentColor);
}
</style>
