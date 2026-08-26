<script setup lang="ts">
import { RouterLink, useRoute } from 'vue-router'
import { computed, ref, watch } from 'vue';
import router from '@/router';
import { useUserStore } from '@/stores/poems';
defineProps<{
}>()

const isMobileOpen = ref(false)

const userStore = useUserStore()

const isRegister = computed(() => {
  return !!userStore.user?.username;
});


const route = useRoute()

const toggleMobileMenu = () => {
  isMobileOpen.value = !isMobileOpen.value
}

const searchTerm = ref("")
const debouncedSearchTerm = ref<string>('')
let timeoutId: any = null


watch(route, () => {
  searchTerm.value = ""
  isMobileOpen.value = false

})


watch(searchTerm, (newVal) => {
  clearTimeout(timeoutId)
  timeoutId = setTimeout(() => {
    debouncedSearchTerm.value = newVal;

  }, 300)
})

const sendSearch = () => {
  if (debouncedSearchTerm) {
    router.push({
      name: 'searchPage',
      params: { term: debouncedSearchTerm.value }
    })
  }
}

const sendSearchDirect = () => {
  if (searchTerm) {
    router.push({
      name: 'searchPage',
      params: { term: searchTerm.value }
    })
  }
}

function addPoem() {
  router.push({ name: 'addPoem', params: { id: null } })
}


</script>

<template>

  <header>
    <div class="header-wrapper">
      <RouterLink to="/">
        <div class="logo">✒<span>L</span>at</div>
      </RouterLink>
      <div class="buttons desktop">
        <div class="search-wrapper">
          <input v-model.trim="searchTerm" type="text" placeholder="Search poem..." @keyup.enter="sendSearchDirect"
            maxlength="15"></input>
          <button class="search-btn" @click="sendSearch">🔍︎

          </button>
        </div>
        <a @click="addPoem"> <button class="reverse" aria-label="Add a poem" title="Add a poem">Add poem</button>
        </a>
        <RouterLink v-if="!isRegister" to="/register"><button class="reverse sign-up" aria-label="Sign up"
            title="Sign up">Sign up</button></RouterLink>
        <RouterLink v-else to="/account"><button class="reverse" aria-label="Go to account"
            title="Account">Account</button></RouterLink>

        <RouterLink to="/poems"> <button class="reverse all-poems" aria-label="See all poems" title="See all poems">See
            all</button></RouterLink>
      </div>

      <button class="mobileMenu" @click="toggleMobileMenu"> ≡</button>
      <div v-if="isMobileOpen" class="mobile-menu-wrapper">
        <div class="buttons">

          <a @click="addPoem"> <button class="reverse" aria-label="Add a poem" title="Add a poem">Add poem</button>
          </a>
          <RouterLink v-if="!isRegister" to="/register"><button class="reverse sign-up" aria-label="Sign up"
              title="Sign up">Sign up</button></RouterLink>
          <RouterLink v-else to="/account"><button class="reverse" aria-label="Go to account"
              title="Account">Account</button></RouterLink>


          <RouterLink to="/poems"> <button class="reverse" aria-label="See all poems" title="See all poems">All
              poems</button>
          </RouterLink>
          <div class="search-wrapper">
            <input v-model.trim="searchTerm" type="text" placeholder="Search poem..." @keyup.enter="sendSearchDirect"
              maxlength="15"></input>
            <button class="search-btn" @click="sendSearch" aria-label="search" title="search">🔍︎

            </button>
          </div>

          <RouterLink to="/">
            <div class="logo">✒<span>L</span>at</div>
          </RouterLink>

        </div>
      </div>

    </div>

  </header>



</template>

<style scoped>
header {
  width: 100%;
  padding: var(--defaultSmallPadding);
  z-index: 9999;
}

header>div {
  max-width: var(--containerDefaultWidth);
  width: 100%;
  margin: auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: var(--defaultLight);
  border: 1px solid var(--defaultDark);
  border-radius: 50px;
  padding: 5px 20px;
}

header {
  position: fixed;
}

header .buttons {
  display: flex;
  gap: var(--defaultSmallPadding);
}


header .logo {
  font-size: 40px;
}

header .buttons button {
  padding: 5px 10px;
  background-color: transparent;
  color: var(--defaultDark);
  border: none;
  transition: var(--transitionTime) all;


}

header .buttons button.search-btn {
  max-width: unset;
  width: unset;
  padding-left: 5px;
  padding-right: 5px;
}


header .buttons a {
  display: flex;
  justify-content: center;
  align-items: center;
}


header input {
  margin-right: 5px;
}

header button.mobileMenu {
  display: none;
}

header .mobile-menu-wrapper {
  display: none;
}

header .search-wrapper {
  display: flex;
}

header .search-wrapper input {
  max-width: 250px;
  width: 100%;
}

@media(max-width:734px) {
  header .buttons.desktop {
    display: none;
  }

  header button.mobileMenu {
    display: block;
    padding: 5px 45px;
    font-size: 25px;
  }

  header .mobile-menu-wrapper {
    display: block;
    padding: var(--defaultPagePadding);
    width: 100%;
    max-width: 300px;
    background-color: var(--defaultLight);
    position: absolute;
    top: var(--headerHeight);
    right: var(--defaultSmallPadding);
    border: 1px solid var(--defaultDark);
    border-radius: 5px;
  }

  header .mobile-menu-wrapper .buttons {
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: flex-start;
    gap: var(--defaultSmallPadding);

  }

}
</style>
