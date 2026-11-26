<script setup lang="ts">
import Poem from './Poem.vue';
import { onMounted, ref, watch } from 'vue';
import Loader from './Loader.vue';
import Error from './Error.vue';

export interface Poem {
  _id: string
  title: string
  text: string
  author: string
  main_category: string
  tags: string[]
  categories: string[]
}

const searchTerm = ref('');
const favouritePoems = ref<Poem[]>([]);
const favouritePoemsRaw = ref<Poem[]>([]);
const isLoading = ref(false);
const isError = ref(false);

async function fetchFavouritePoems() {
  isLoading.value = true;
  isError.value = false;

  try {
    const response = await fetch('/api/favouritePoems', {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json'
      }
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();

    const list: Poem[] = Array.isArray(data)
      ? data
      : (data.poems ?? data.favourites ?? []);

    favouritePoemsRaw.value = list;
    applySearchFilter();
  } catch (e) {
    console.error('Error fetching favourite poems:', e);
    isError.value = true;
    favouritePoems.value = [];
  } finally {
    isLoading.value = false;
  }
}

function applySearchFilter() {
  const query = searchTerm.value.toLowerCase().trim();

  if (!query) {
    favouritePoems.value = [...favouritePoemsRaw.value];
    return;
  }

  favouritePoems.value = favouritePoemsRaw.value.filter((poem) => {
    const matchesTitle = poem.title?.toLowerCase().includes(query);
    const matchesAuthor = poem.author?.toLowerCase().includes(query);
    const matchesMainCategory = poem.main_category?.toLowerCase().includes(query);
    const matchesTags = Array.isArray(poem.tags) &&
      poem.tags.some((tag) => tag.toLowerCase().includes(query));
    const matchesCategories = Array.isArray(poem.categories) &&
      poem.categories.some((cat) => cat.toLowerCase().includes(query));

    return (
      matchesTitle ||
      matchesAuthor ||
      matchesMainCategory ||
      matchesTags ||
      matchesCategories
    );
  });
}

onMounted(() => {
  fetchFavouritePoems();
});

watch(searchTerm, () => {
  applySearchFilter();
});
</script>

<template>
  <div class="page-wrapper">
    <div class="title-wrapper">
      <p class="title">Your favourite poems</p>
      <input
        v-model="searchTerm"
        type="text"
        placeholder="Search in favourites..."
        class="search-input"
      />
    </div>

    <Loader v-if="isLoading" />

    <div v-else class="all-wrapper">
      <Error v-if="isError" />

      <div v-else-if="favouritePoems.length" class="wrapper">
        <div
          v-masonry
          transition-duration="300ms"
          fit-width="true"
          item-selector=".item"
          class="masonry-container masonryWrap"
          gutter="20"
        >
          <div
            v-masonry-tile
            class="item"
            v-for="poem in favouritePoems"
            :key="poem._id"
          >
            <Poem :poem="poem" />
          </div>
        </div>
      </div>

      <div v-else class="empty-state">
        <p v-if="searchTerm.trim()">
          No favourite poems found for "{{ searchTerm.trim() }}".
        </p>
        <p v-else>
          You don't have any favourite poems yet.
        </p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.all-wrapper .wrapper {
  max-width: var(--containerDefaultWidth);
  width: 100%;
  margin: auto;
}

.all-wrapper .masonryWrap {
  margin: auto;
}

.all-wrapper .masonryWrap .item {
  padding-top: 20px;
}

.title-wrapper{
  display: flex;
  flex-direction: column;
}

.title-wrapper .title {
  font-size: 3rem;
}

.search-input {
  margin-top: 12px;
  padding: 10px 14px;
  width: 100%;
  max-width: 400px;
  border-radius: 8px;
  border: 1px solid #ccc;
  font-size: 1rem;
}

.empty-state {
  text-align: center;
  padding: 40px 16px;
}
</style>