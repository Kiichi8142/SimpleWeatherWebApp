<script setup>
import axios from 'axios'
import { useRoute } from 'vue-router'
import { ref, onMounted } from 'vue'

const route = useRoute()

const items = ref()
const apiKey = import.meta.env.VITE_WEATHER_API_KEY

const baseURL = "http://api.weatherapi.com/v1/"

function fetchSearchWeather() {
    const params = {
        key: apiKey,
        q: '18.895430531790694, 99.01034697034198',
    }
    axios.get(baseURL + "forecast.json", { params }).then((response) => {
        console.log(response.data)
        items.value = response.data
    }).catch((err) => {
        console.log(err);
    })
}

onMounted(fetchSearchWeather)

</script>

<template>
    <div class="text-white flex flex-col items-center justify-center">
        <div v-if="items" class="flex flex-col max-w-3xl w-full">
            <p class="p-4 font-semibold self-center text-4xl">A weather forecast in {{
                items.forecast.forecastday.length }} days</p>
            <p class="px-4 font-medium text-xl">#{{ items.location.name }}, {{ items.location.country }}</p>
            <p class="text-gray-400 px-4">Forecast Weather</p>
            <div class="m-4 p-4 flex items-center border rounded-md gap-x-8 hover:bg-neutral-700"
                v-for="(data, key) in items.forecast.forecastday.slice().reverse()" v-bind:key="key">
                <img :src="data.day.condition.icon" alt="">
                <div class="flex flex-col md:flex-row gap-x-8">
                    <p>Date: {{ data.date }}</p>
                    <p>Avg temp: {{ data.day.avgtemp_c }}</p>
                    <p>Condition: {{ data.day.condition.text }}</p>
                </div>
            </div>
        </div>
    </div>
</template>
