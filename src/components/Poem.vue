<script setup lang="ts">
import { useRouter } from 'vue-router'
import PoemSymbol from './PoemSymbol.vue';
import { useUiStore, useUserStore } from '@/stores/poems.ts';
import { computed } from 'vue';


const props = defineProps<{

  poem: {
    _id:any,
    title: string
    text: string
    tags: string[]
    main_category: string
    categories: string[]
    author?: string
  }
}>()

const router = useRouter()
const uiStore = useUiStore()

const openSideMenu = () => {
  uiStore.openMenu(props.poem)
}


function searchPage(term: string) {
  router.push({
    name: 'searchPage',
    params: { term: term }
  })
}

const userStore = useUserStore()

const isRegister = computed(() => {
  return !!userStore.user?.username; 
});

const isAuthor = computed(() =>{
  return isRegister && userStore.user?.username === props.poem.author
})


function editPoem(){
  router.push({ name: 'edit-poem', params: { id: props.poem._id } })
}

function decodeText(text?: string): string {
  if (!text) return ''

let cleanText = text
    .replace(/^['"]|['"]$/g, '')
    .replace(/\\'/g, "'")
    .replace(/\\"/g, '"')
    .replace(/\\n/g, '\n')

  return cleanText
    .split('\n')
    .filter((line) => line.trim() !== '')
    .map((line) => `<p>${line.trim()}</p>`)
    .join('')
}

</script>

<template>

  <div class="poem container">
    <div v-if="isAuthor" class="edit-button" @click="editPoem">🖍</div>
    <div class="poem-content" @click="openSideMenu">
      <p class="subtitle">{{ poem.title }}</p>
      <div class="content" v-html="decodeText(poem?.text)"></div>
    </div>
    <div class="poem-footer">
      <div class="wrapper">
        <p class="author note">Written by <span class="author-name" @click="searchPage(poem.author || '')"> {{
          poem.author }}</span> in
          <span @click="searchPage(poem.main_category || '')" class="main_category">{{
            poem.main_category }}</span>
        </p>

        <div class="tags note">
          <p>Tags: </p>
          <div>
            <div v-for="tag in poem.tags.slice(0, 6)" :key="tag" @click="searchPage(tag)">
              #{{ tag }}
            </div>
          </div>
        </div>

      </div>

      <PoemSymbol :category="poem.main_category"></PoemSymbol>
    </div>
  </div>


</template>

