<script setup lang="ts">
import {ref, onMounted, computed} from 'vue';
import { resources } from '../data/sample-resources';


const visibleCount = ref(6);
const selectedTag = ref<string[]>(['ALL']);
const isDark = ref(false);

const toggleBookmark = (res: any) => {
  const resource = resources.find(r => r.title === res.title);
  if (resource) {
    resource.isBookmarked = !resource.isBookmarked;
  }
};

const categoryTags = computed(() => {
  const tags = new Set<string>();
  resources.forEach(res => {
    if (res.category) tags.add(res.category);
  });
  return Array.from(tags).sort();
});

const toggleTag = (tag: string) => {
  const allTags = categoryTags.value;
  if (tag === 'ALL') {
    if (selectedTag.value.includes('ALL')) {
      selectedTag.value.length = 0;

    }
  }
  if (selectedTag.value.includes(tag)) {
    selectedTag.value = selectedTag.value.filter(t => t !== tag);
    } else {
      selectedTag.value.push(tag);
    }
  selectedTag.value = selectedTag.value.filter(t => t !== 'ALL');
  if (selectedTag.value.length === 10) {
    selectedTag.value = ['ALL'];
  }
};
const filteredResources = computed(() => {
  const filtered = selectedTag.value.includes('ALL')
  ? resources : resources.filter(res =>selectedTag.value.includes(res.category));
return filtered.slice(0, visibleCount.value);
});
 const totalFilteredCount = computed(() => {
  return selectedTag.value.includes('ALL')
    ? resources.length
    : resources.filter(res => selectedTag.value.includes(res.category)).length;
});

const toggleDarkMode = () => {
  isDark.value = !isDark.value;
  document.documentElement.classList.toggle('dark', isDark.value);
};
onMounted(() => {
  const savedMode = localStorage.getItem('darkMode');
  isDark.value = savedMode === 'true';
  document.documentElement.classList.toggle('dark', isDark.value);
});

</script>

<template>
  <section class="resources">

    <div class="header-row">
      <h2> Your Resources </h2>
      <div class="header-icons">
        <span class="material-symbols-outlined" title="Toggle Dark Mode" @click="toggleDarkMode">
          {{ isDark ? 'light_mode' : 'dark_mode' }}
        </span>
        <span class="material-symbols-outlined" title="Edit" @click="">edit</span>
        <span class="material-symbols-outlined" title="Add" @click="">add_2</span>
        <span class="material-symbols-outlined" title="List" @click="">format_list_bulleted</span>
        <span class="material-symbols-outlined" title="Sort" @click="">sort</span>
    </div>
    </div>
    <p> View and bookmark resources. </p>
    <div class="filter-tag">
      <button
        v-for="tag in ['ALL', ...categoryTags]"
        :key="tag"
        :class="{ active: selectedTag.includes(tag) }"
        @click="toggleTag(tag)">
        {{ tag }}
      </button>
    </div>
    <div class="grid">
      <div class="card" v-for="(res, index) in filteredResources"
      :key="index"
      :style="`background-image: url('/images/${res.img}.jpg')`">
        <div class="card-content">
        <p class="category">
          <span class="material-symbols-outlined">emoji_objects</span>
          {{ res.category }} - Resources </p>
        <p class="title">
          {{ res.title }}</p>
          <span class ="material-symbols-outlined bookmark-icon"
          :class = "{bookmarked: res.isBookmarked}"
          @click="toggleBookmark(res)"
          title="Bookmark" >
        {{ res.isBookmarked ? 'bookmark' : 'bookmark_border'}}
          </span>
        </div>
      </div>
    </div>
    <div v-if="visibleCount < totalFilteredCount" class="show-more">
      <button @click="visibleCount += 6">Show More</button>
    </div>
  </section>
</template>


<style scoped>
.resources {
  max-width: 800px;
  margin: 1rem auto;
  font-family: helvetica, Arial, sans-serif;
  padding: 1rem;
}
.grid {
  display: grid;
  grid-template-columns: repeat(3,1fr);
  gap: 1.5rem;
}
.card {
  position: relative;
  height: 250px;
  width: 250px;
  background: transparent;
  background-size: contain;
  background-position: center;
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
.card-image {
  width: 100%;
  height: 150px;
  object-fit: contain;
}
.card-content {
  padding: 1rem;
  position: absolute;
  bottom: 1rem;
  left: 1rem;
}
.category {
  font-size: .8rem;
  color: white;
}
.title {
  font-size: .8rem;
  margin-top: 0.5rem;
  color: white;
}
.card:hover {
  transform: translateY(-5px);
  transition: transform 0.2s;
}

.card:hover .card-image {
  filter: brightness(0.9);
}
.material-symbols-outlined {
  font-variation-settings: 'FILL' 0, 'wght' 400, 'GRAD' 0, 'opsz' 24;
  animation: pulse 2s infinite;
  }

@keyframes pulse {
    0% {
      transform: scale(1);
      opacity: 1;
    }
    50% {
      transform: scale(1.05);
      opacity: 0.8;
    }
    100% {
      transform: scale(1);
      opacity: 1;
    }
  }

.filter-tag {
  gap: 1rem;
  margin-bottom: 1rem;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  padding: 1rem;

}
.filter-tag button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  background-color: #fcf6f6;
  color: #333;
  cursor: pointer;
  transition: background-color 0.3s, color 0.3s;
}
.filter-tag button.active {
  background-color: #333;
  color: #fcf6f6;
}
.filter-tag button:hover {
  background-color: #ddd;
  color: #000;
}
.header-icons {
  display: flex;
  gap: 1rem;

}
.header-icons span {
  font-size: 1.5rem;
  cursor: pointer;
  color: #333;
  transition: color 0.3s;
}
.header-icons span:hover {
  color: #007bff;
}
.header-icons span:active {
  color: #0056b3;
}
.header-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}
.cursor-pointer {
  cursor: pointer;
}
.show-more {
  text-align: center;
  margin-top: 1rem;

}
.show-more button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  background-color: #d3d3d3;
  color: rgb(2, 2, 2);
  cursor: pointer;
  transition: background-color 0.3s, color 0.3s;
}
.show-more button:hover {
  background-color: #a3a3a3;
  color: #fff;
}
</style>
<style>
:root{
  --bg-color: #f0f0f0;
  --text-color: #333;
  --primary-color: #007bff;
  --secondary-color: #6c757d;
}
body, html {
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
  background-color: var(--bg-color);
  color: var(--text-color);
}
.dark {
  --bg-color: #121212;
  --text-color: #e0e0e0;
  --primary-color: #bb86fc;

}
.dark .header-icons span {
  color: var(--text-color);
}
.dark .show-more button {
  background-color: var(--secondary-color);
  color: var(--text-color);
}

.bookmark-icon {
  position: relative;
  top: 1rem;
  right: 1rem;
  font-size: 2rem;
  color: white;
  cursor: pointer;
  transition: color 0.3s;
  z-index: 2;
}
.bookmark-icon.bookmarked {
  color: orange;
}
</style>