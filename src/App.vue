<script setup>
import { ref, computed, onMounted, watch } from "vue";
import axios from "axios";

const photos = ref([]);
const albumInput = ref(localStorage.getItem("albumInput") || "");
const loading = ref(false);
const page = ref(1);
const perPage = 30;
const sortBy = ref("id");
const sortOrder = ref("asc");
const darkMode = ref(localStorage.getItem("darkMode") === "true");

const fetchPhotos = async () => {
  loading.value = true;
  try {
    const albumIds = albumInput.value.trim() ? `?${albumInput.value.trim().split(" ").map(id => `albumId=${id}`).join("&")}` : "";
    const { data } = await axios.get(`https://jsonplaceholder.typicode.com/photos${albumIds}`);
    photos.value = data;
  } catch (error) {
    console.error(error);
  } finally {
    loading.value = false;
  }
};

const sortedPhotos = computed(() => {
  return [...photos.value]
    .sort((a, b) => {
      if (sortBy.value === "id" || sortBy.value === "albumId") {
        return sortOrder.value === "asc" ? a[sortBy.value] - b[sortBy.value] : b[sortBy.value] - a[sortBy.value];
      }
      return sortOrder.value === "asc" ? a[sortBy.value].localeCompare(b[sortBy.value]) : b[sortBy.value].localeCompare(a[sortBy.value]);
    })
    .slice(0, page.value * perPage);
});

const loadMore = () => page.value++;

const toggleSort = (column) => {
  sortBy.value = column;
  sortOrder.value = sortBy.value === column && sortOrder.value === "asc" ? "desc" : "asc";
};

const toggleTheme = () => {
  darkMode.value = !darkMode.value;
  localStorage.setItem("darkMode", darkMode.value);
};

watch(albumInput, (newVal) => {
  localStorage.setItem("albumInput", newVal);
});

onMounted(fetchPhotos);
</script>

<template>
  <div :class="{'dark bg-gray-900 text-black': darkMode, 'bg-white text-gray-900': !darkMode}" class="min-h-screen p-6">
    <div class="max-w-5xl mx-auto p-6 rounded-lg shadow-lg border bg-gray-100 dark:bg-gray-800">
      <div class="flex gap-4 mb-6">
        <input v-model="albumInput" placeholder="ID альбомов (через пробел)" class="border p-3 rounded-lg w-full shadow-md focus:ring-2 focus:ring-blue-400 dark:bg-gray-700 dark:border-gray-600" />
        <button @click="fetchPhotos" class="bg-blue-600 text-white px-6 py-3 rounded-lg hover:bg-blue-500 focus:ring-2 focus:ring-blue-300">Поиск</button>
        <button @click="toggleTheme" class="bg-gray-600 text-white px-4 py-3 rounded-lg hover:bg-gray-500">🌙/☀</button>
      </div>
      <div v-if="loading" class="my-4 text-center text-gray-500">Загрузка...</div>
      <div class="overflow-x-auto">
        <table class="w-full table-auto border-collapse">
          <thead class="bg-gray-200 dark:bg-gray-700 sticky top-0">
            <tr class="text-left text-gray-700 dark:text-black">
              <th @click="toggleSort('id')" class="cursor-pointer px-4 py-2 border-b hover:bg-gray-300 dark:hover:bg-gray-600">Ид</th>
              <th @click="toggleSort('albumId')" class="cursor-pointer  px-4 py-2 border-b hover:bg-gray-300 dark:hover:bg-gray-600">Альбом</th>
              <th @click="toggleSort('title')" class="cursor-pointer  px-4 py-2 border-b hover:bg-gray-300 dark:hover:bg-gray-600">Название</th>
              <th class="px-4 py-2 border-b">Ссылка</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="photo in sortedPhotos" :key="photo.id" class="hover:bg-gray-50 dark:hover:bg-gray-600">
              <td class="px-4 py-2 border-b">{{ photo.id }}</td>
              <td class="px-4 py-2 border-b">{{ photo.albumId }}</td>
              <td class="px-4 py-2 border-b truncate" :title="photo.title">{{ photo.title }}</td>
              <td class="px-4 py-2 border-b"><a :href="photo.url" target="_blank" class="text-blue-500 hover:text-blue-700">Ссылка</a></td>
            </tr>
          </tbody>
        </table>
      </div>
      <div v-if="sortedPhotos.length < photos.length" class="mt-4">
        <button @click="loadMore" class="w-full bg-gray-500 text-white py-3 rounded-lg hover:bg-gray-600 focus:ring-2 focus:ring-gray-300">Загрузить ещё</button>
      </div>
    </div>
  </div>
</template>
