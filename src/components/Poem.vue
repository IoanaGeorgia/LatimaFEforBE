<script setup lang="ts">
import { useRouter } from 'vue-router'
import PoemSymbol from './PoemSymbol.vue'
import { useUiStore, useUserStore } from '@/stores/poems.ts'
import { computed } from 'vue'

const props = defineProps<{
  poem: {
    _id: any
    title: string
    text: string
    tags: string[]
    main_category: string
    categories: string[]
    author?: string
    isFavourite?: boolean
  }
}>()

const router = useRouter()
const uiStore = useUiStore()
const userStore = useUserStore()

const openSideMenu = () => {
  uiStore.openMenu(props.poem)
}

function searchPage(term: string) {
  router.push({
    name: 'searchPage',
    params: { term: term }
  })
}

const isRegister = computed(() => {
  return !!userStore.user?.username
})

const isAuthor = computed(() => {
  return isRegister.value && userStore.user?.username === props.poem.author
})

const favouriteIdsSet = computed(() => {
  const list = userStore.user?.isFavourite ?? []
  return new Set(list.map((id: any) => String(id)))
})

const isPoemFavourite = computed(() => {
  return favouriteIdsSet.value.has(String(props.poem._id))
})

async function toggleFavourite(id: any) {
  try {
    const response = await fetch('/api/addToFavourites', {
      method: 'POST',
      credentials: 'include',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ id: String(id) })
    })

    if (!response.ok) throw new Error(`HTTP error! Status: ${response.status}`)

    const result = await response.json()

    if (result.success) {
      if (userStore.user) {
        if (Array.isArray(result.favourites)) {
          userStore.user.isFavourite = result.favourites
        } else {
          const current = (userStore.user.isFavourite ?? []).map((item: any) => String(item))
          const target = String(id)
          userStore.user.isFavourite = current.includes(target)
            ? current.filter((item) => item !== target)
            : [...current, target]
        }
      }

      if (uiStore.activePoem?._id && String(uiStore.activePoem._id) === String(id)) {
        uiStore.setActivePoemFavourite()
      }
    }
  } catch (error) {
    console.error('Failed to edit poem:', error)
  }
}

function editPoem() {
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
        <div v-if="isRegister" class="favourite" @click.stop="toggleFavourite(poem._id)">
          <p v-if="isPoemFavourite"><span class="favoured">★</span></p>
          <p v-else><span>☆</span></p>
        </div>
        <p class="author note">Written by <span class="author-name" @click="searchPage(poem.author || '')">{{
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

<style scoped>
.favourite {
  cursor: pointer;
  font-size: 2rem;
}

.favourite .favoured {
  color: var(--accentColor);
}</style>