<script setup lang="ts">
import { useRouter } from 'vue-router';
import PoemSymbol from './PoemSymbol.vue';
import { useUiStore } from '@/stores/poems';

const uiStore = useUiStore();
const router = useRouter();

function searchPage(term: string) {
  if (!term) return;

  router.push({
    name: 'searchPage',
    params: { term: term }
  });

  uiStore.closeMenu();
}
</script>

<template>
  <div v-if="uiStore.isSideMenuOpen && uiStore.activePoem" class="sidemenuMainWrapper">
    <div class="sidemenu">
      <button class="closeSideMenu" @click="uiStore.closeMenu">✕</button>
      <div class="sidemenu-wrapper">
        <div class="poem container">
          <p class="subtitle">{{ uiStore.activePoem.title }}</p>
          <div class="content" v-html="uiStore.activePoem.text"></div>

          <div class="poem-footer">
            <div class="wrapper">
              <p class="author note">
                Written by
                <span class="author-name" @click="searchPage(uiStore.activePoem.author)">{{ uiStore.activePoem.author ||
                  'Anonymous' }}</span>
                in <span class="main_category" @click="searchPage(uiStore.activePoem.main_category)">{{ uiStore.activePoem.main_category }}</span>
              </p>



              <div v-if="uiStore.activePoem.categories.length" class="tags note">
                <p>Categories: </p>
                <p class="cat_tags">
                  <div v-for="category in uiStore.activePoem.categories" :key="category">
                    <span @click="searchPage(category)">{{ category }}</span>
                  </div>
                </p>
              </div>

              <div v-if="uiStore.activePoem.tags.length" class="tags note">
                <p>Tags: </p>
                <div>
                  <div v-for="tag in uiStore.activePoem.tags?.slice(0, 10)" :key="tag" @click="searchPage(tag)">
                    #{{ tag }}
                  </div>
                </div>
              </div>
            </div>

            <PoemSymbol :category="uiStore.activePoem.main_category" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.sidemenuMainWrapper {
  position: fixed;
  top:0px;
  right: 0px;
  bottom: 0px;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 99999;
  width: 100%;
  padding:var(--defaultSmallPadding);
  background-color: rgba(0, 0, 0, 0.51);
  overflow:hidden;
}

.sidemenu {
  background-color: var(--defaultLight);
  border: 1px solid var(--defaultDark);
  max-width: 700px;
  width: 100%;
  z-index: 30;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: flex-start;
  position: relative;
  max-height:600px;
  height:100%;
  border-radius:20px;
  padding-right:20px;

}

.sidemenu .closeSideMenu {
  position: absolute;
  top: var(--defaultSmallPadding);
  right: var(--defaultSmallPadding);
  padding: 5px 45px;
}


.sidemenu .sidemenu-wrapper {
  flex-grow: 1;
  overflow-y: auto;
  width: 100%;
  padding: var(--defaultMargin);
  padding-right: 0px;
  padding-top:0px;
  margin-top: 60px;
}

.container.poem {
  max-width: 500px;
  width: 100%;
  height: fit-content;
  margin: auto;
  display: flex;
  flex-direction: column;
  min-height: 100%;
  min-width: unset;
}

.poem .content {
  flex-grow: 1;

}

@media(max-width:595px) {
  .sidemenu {
    padding-right: 0px;


  }

  .sidemenu .sidemenu-wrapper {
    padding-right: var(--defaultMargin);
  }

  .poem-footer {
    flex-direction: column-reverse;
    justify-content: flex-start;
    align-items: flex-start;
  }


  .poem-footer .symbol-wrapper {
    width: 100%;
    text-align: center;
    min-height: 50px;
  }

  .symbol-wrapper div {
    line-height: 50px;
  }

  .poem .content {
    margin-bottom: var(--defaultSmallPadding)
  }
}
</style>
