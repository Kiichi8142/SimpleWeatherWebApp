<script setup>
import { RouterLink, RouterView } from 'vue-router'
import axios from 'axios'
import { ref, onMounted } from 'vue'

const items = ref()
const apiKey = import.meta.env.VITE_WEATHER_API_KEY

const baseURL = "https://api.weatherapi.com/v1/"

function fetchWeather() {
  const params = {
    key: apiKey,
    q: '18.895430531790694, 99.01034697034198',
  }
  axios.get(baseURL + "current.json", { params }).then((response) => {
    items.value = response.data
  }).catch((err) => {
    console.log(err);
  })
}

onMounted(fetchWeather)
</script>

<template>
  <header class="flex flex-col p-8 text-white items-center">
    <div v-if="items" class="flex items-center m-8 gap-x-4">
      <img alt=" Vue logo" class="" :src="items.current.condition.icon" width="64" height="64">
      <p class="font-medium text-xl">A simple weather website</p>
    </div>
    <div>
      <nav class="flex flex-row gap-8">
        <RouterLink class="p-4 font-medium hover:border-b hover:border-sky-500" active-class="p-4 border-b border-sky-500"
          to="/">Current
        </RouterLink>
        <RouterLink class="p-4 font-medium hover:border-b hover:border-sky-500" active-class="p-4 border-b border-sky-500"
          to="/history">History</RouterLink>
        <RouterLink class="p-4 font-medium hover:border-b hover:border-sky-500" active-class="p-4 border-b border-sky-500"
          to="/forecast">Forecast
        </RouterLink>
      </nav>
    </div>
  </header>

  <RouterView />
</template>