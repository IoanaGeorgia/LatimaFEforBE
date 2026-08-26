<script setup lang="ts">
import { useCategoriesStore, usePoemStore, useUserStore } from '@/stores/poems'
import { ref, reactive, computed, watch } from 'vue'
import TopRead from './TopRead.vue'
import Loader from './Loader.vue'
import DOMPurify from 'dompurify'

function encodeForInput(text?: string): string {
  if (!text) return ''
  return text
    .replace(/^['"]|['"]$/g, '')
    .replace(/\\n/g, '\n')
    .replace(/\\'/g, "'")
    .replace(/\\"/g, '"')
}

function decodeFromInput(text: string): string {
  if (!text) return ''
  return text.replace(/\n/g, '\\n')
}

const props = defineProps<{
  id?: string | number
}>()

const poemsStore = usePoemStore()
const categoriesStore = useCategoriesStore()

export interface Poem {
  id: any
  title: string
  text: string
  author: string
  main_category: string
  tags: string[]
  categories: string[]
}

interface Category {
  id: string | number
  name: string
  symbol: string
}

const userStore = useUserStore()

const categories = categoriesStore.categories as unknown as Category[]

const poemProp = computed(() =>
  props.id ? poemsStore.poems.find((poem) => String(poem._id) === String(props.id)) : null
)

let poem = ref({
  title: poemProp.value?.title ?? '',
  text: encodeForInput(poemProp.value?.text),
  tags: poemProp.value?.tags ? [...poemProp.value.tags] : [],
  categories: poemProp.value?.categories ? [...poemProp.value.categories] : [],
  main_category: poemProp.value?.main_category ?? '',
  author: userStore.user?.username
})

let isOpenMain = ref(false)
let catDropdownMainValue = ref('Select main category')
let isOpen = ref(false)
let catDropdownValue = ref('Select categories')
let isLoading = ref(false)
let isSubmitted = ref(false)

const errors = reactive({
  title: '',
  text: '',
  main_category: '',
  categories: '',
  tags: '',
  submit: ''
})

const clearErrors = () => {
  errors.title = ''
  errors.text = ''
  errors.main_category = ''
  errors.categories = ''
  errors.tags = ''
  errors.submit = ''
}

const validateForm = (): boolean => {
  clearErrors()
  let isValid = true

  if (!poem.value.title.trim()) {
    errors.title = 'Title is required.'
    isValid = false
  }

  if (!poem.value.text.trim()) {
    errors.text = 'Poem body cannot be empty.'
    isValid = false
  }

  if (!poem.value.main_category) {
    errors.main_category = 'Please select a main category.'
    isValid = false
  }

  if (poem.value.categories.length === 0) {
    errors.categories = 'Select at least one category.'
    isValid = false
  }

  if (poem.value.tags.length === 0) {
    errors.tags = 'Please add at least one tag.'
    isValid = false
  }

  return isValid
}

const autoResize = (event: Event) => {
  const el = event.target as HTMLTextAreaElement
  if (el) {
    el.style.height = 'auto'
    el.style.height = el.scrollHeight + 'px'
  }
}

const addTags = (event: KeyboardEvent) => {
  const target = event.target as HTMLInputElement
  const val = target?.value.trim()
  if (val && val.length > 2) {
    if (poem.value.tags.length >= 10) {
      errors.tags = 'Maximum of 10 tags allowed.'
      return
    }
    let tempSet = new Set(poem.value.tags)
    tempSet.add(val)
    poem.value.tags = [...tempSet]
    target.value = ''
    errors.tags = ''
  }
}

const toggleMainDropdown = () => {
  isOpenMain.value = !isOpenMain.value
}

const selectMainCategory = (name: string | undefined) => {
  if (name) {
    catDropdownMainValue.value = name
    poem.value.main_category = name
    isOpenMain.value = false
    errors.main_category = ''
  }
}

const toggleDropdown = () => {
  isOpen.value = !isOpen.value
}

const selectCategory = (name: string | undefined) => {
  if (name) {
    catDropdownValue.value = name
    let tempSet = new Set(poem.value.categories)
    tempSet.add(name)
    poem.value.categories = [...tempSet]
    isOpen.value = false
    errors.categories = ''
  }
}

const deleteCat = (name: string | undefined) => {
  if (name) {
    poem.value.categories = poem.value.categories.filter((cat: string) => cat !== name)
  }
}

const deleteTag = (name: string | undefined) => {
  if (name) {
    poem.value.tags = poem.value.tags.filter((tag: string) => tag !== name)
  }
}

const submitForm = async () => {
  if (!validateForm()) return

  isLoading.value = true

  const sanitizedText = DOMPurify.sanitize(poem.value.text, { ALLOWED_TAGS: [] })

  const payload = {
    ...poem.value,
    text: decodeFromInput(sanitizedText) 
  }


  if (props.id) {
    try {
      const response = await fetch('/api/editPoem', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ ...payload, _id: props.id })
      })

      if (!response.ok) throw new Error(`HTTP error! Status: ${response.status}`)

      const result = await response.json()
      poemsStore.updatePoem(result.data)
      isSubmitted.value = true
    } catch (error) {
      console.error('Failed to edit poem:', error)
      errors.submit = 'Something went wrong editing your poem. Please try again.'
    } finally {
      isLoading.value = false
    }
  } else {
    try {
      const response = await fetch('/api/addPoem', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      })

      if (!response.ok) throw new Error(`HTTP error! Status: ${response.status}`)

      const result = await response.json()
      poemsStore.addPoem(result.data)
      isSubmitted.value = true
    } catch (error) {
      console.error('Failed to submit poem:', error)
      errors.submit = 'Something went wrong submitting your poem. Please try again.'
    } finally {
      isLoading.value = false
    }
  }
}

function getCategorySymbol(categoryName: string) {
  if (!categories || !Array.isArray(categories)) return ''
  const found = categories.find(
    (cat: Category) => cat.name.toLowerCase() === categoryName?.toLowerCase()
  )
  return found ? found.symbol : ''
}

watch(
  () => props.id,
  () => {
    const currentPoem = poemProp.value
    if (currentPoem) {
      poem.value = {
        title: currentPoem.title ?? '',
        text: encodeForInput(currentPoem.text),
        tags: Array.isArray(currentPoem.tags) ? [...currentPoem.tags] : [],
        categories: Array.isArray(currentPoem.categories) ? [...currentPoem.categories] : [],
        main_category: currentPoem.main_category ?? '',
        author: userStore.user?.username
      }
      catDropdownMainValue.value = currentPoem.main_category || 'Select main category'
    } else {
      poem.value = {
        title: '',
        text: '',
        tags: [],
        categories: [],
        main_category: '',
        author: userStore.user?.username
      }
      catDropdownMainValue.value = 'Select main category'
      catDropdownValue.value = 'Select categories'
      isSubmitted.value = false
      clearErrors()
    }
  },
  { immediate: true }
)
</script>

<template>
  <div class="page-wrapper add-poem">
    <div class="title-wrapper">
      <p v-if="props.id" class="title">Edit your poem</p>
      <p v-else class="title">Make your own poem</p>

    </div>

    <div v-if="isSubmitted">
      <Loader v-if="isLoading" />
      <div v-else class="submitted">
        <p v-if="props.id"><strong>Your poem has been successfully edited!</strong></p>
        <p v-else><strong>Thank you for your submission!</strong></p>
        <p v-if="props.id">You can now go back to the main page.</p>
        <p v-else>We will review it in 3-5 working days and let you know further ahead if there are any issues.</p>

      </div>
    </div>

    <div v-else class="form-wrapper">
      <form class="container" @submit.prevent="submitForm">

        <div class="input-wrapper">
          <label>Poem title:</label>
          <input type="text" v-model="poem.title" :class="{ 'has-error': errors.title }"
            placeholder="Write the title here..." />
          <p v-if="errors.title" class="error-msg">{{ errors.title }}</p>
        </div>

        <div class="input-wrapper">
          <label>Poem body: </label>
          <textarea v-model="poem.text" @input="autoResize" :class="{ 'has-error': errors.text }"
            placeholder="Write here the content of your poem..." />
          <p v-if="errors.text" class="error-msg">{{ errors.text }}</p>
        </div>

        <div class="input-wrapper">
          <label>Main category:</label>
          <div class="cat-dropdown" :class="{ 'has-error': errors.main_category }">
            <p class="main-btn" @click="toggleMainDropdown">
              <span>{{ catDropdownMainValue }}</span>
            <div v-if="isOpenMain" class="arrow opened">〉</div>
            <div v-else class="arrow closed">〉</div>
            </p>
            <div v-if="isOpenMain" class="options">
              <div v-for="category in categories" :key="category.id" @click="selectMainCategory(category.name)">
                <span>{{ category.symbol }} {{ category.name }}</span>
              </div>
            </div>
          </div>
          <p v-if="errors.main_category" class="error-msg">{{ errors.main_category }}</p>
        </div>

        <div class="input-wrapper">
          <label>Categories (at least 1): </label>
          <div class="cat-dropdown" :class="{ 'has-error': errors.categories }">
            <p class="main-btn" @click="toggleDropdown">
              <span>{{ catDropdownValue }}</span>
            <div v-if="isOpen" class="arrow opened">〉</div>
            <div v-else class="arrow closed">〉</div>
            </p>
            <div v-if="isOpen" class="options">
              <div v-for="category in categories" :key="category.id" @click="selectCategory(category.name)">
                <span>{{ category.symbol }} {{ category.name }}</span>
              </div>
            </div>
          </div>

          <div class="categories">
            <div v-if="!poem.categories.length" class="empty-hint">No categories selected</div>
            <div class="tag" v-else v-for="name in poem.categories" :key="name">
              <span>{{ getCategorySymbol(name) }} {{ name }}</span>
              <button type="button" class="close-tag" @click="deleteCat(name)">×</button>
            </div>
          </div>
          <p v-if="errors.categories" class="error-msg">{{ errors.categories }}</p>
        </div>

        <div class="input-wrapper">
          <label>Tags (1 to 10 required): </label>
          <input type="text" @keydown.enter.prevent="addTags" :class="{ 'has-error': errors.tags }"
            placeholder="Type tag and press Enter" />
          <div v-if="poem.tags.length > 0" class="input tags">
            <div class="tag" v-for="tag in poem.tags.slice(0, 10)" :key="tag">
              #{{ tag }}
              <button type="button" class="close-tag" @click="deleteTag(tag)">×</button>
            </div>
          </div>
          <p v-if="errors.tags" class="error-msg">{{ errors.tags }}</p>
        </div>

        <div v-if="errors.submit" class="error-msg">
          {{ errors.submit }}
        </div>

        <button type="submit" :disabled="isLoading">
          {{ isLoading ? 'Saving...' : 'Save' }}
        </button>
      </form>
    </div>
    <TopRead />
  </div>
</template>

<style scoped></style>