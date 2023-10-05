<script setup>
import axios from 'axios'
import { ref, onMounted, watch } from 'vue'

const items = ref()
const apiKey = import.meta.env.VITE_WEATHER_API_KEY

const baseURL = "http://api.weatherapi.com/v1/"

const location = ref('Chiang Mai')

function fetchForecastWeather() {
    const params = {
        key: apiKey,
        q: location.value,
        days: 5,
    }
    axios.get(baseURL + "forecast.json", { params }).then((response) => {
        console.log(response.data)
        items.value = response.data
        location.value = '#' + response.data.location.name + ', ' + response.data.location.country
        updateInput(location.value)
    }).catch((err) => {
        console.log(err);
    })
}

function updateInput(formatLocation) {
    const element = document.getElementById('inputLocation')
    if (element) {
        element.value = formatLocation
    }
}

watch(location, (newMessage) => {
    if (!newMessage.includes('#')) {
        if (newMessage) {
            fetchForecastWeather()
        } else {
            location.value = 'Chiang Mai'
        }
    }
})

onMounted(fetchForecastWeather)

</script>

<template>
    <div class="text-white flex flex-col items-center justify-center">
        <div v-if="items" class="flex flex-col w-full max-w-3xl">
            <p class="p-4 font-semibold self-center text-4xl">A weather forecast in {{
                items.forecast.forecastday.length }} days</p>
            <input id="inputLocation" v-model.lazy="location"
                class="bg-neutral-900 focus:bg-neutral-100 focus:text-gray-950 w-auto px-4 font-medium text-xl" type="text">
            <p class="text-gray-400 px-4">Forecast Weather</p>
            <div class="m-4 p-4 flex items-center border rounded-md gap-x-8 hover:bg-neutral-700"
                v-for="(data, key) in items.forecast.forecastday.slice().reverse()" v-bind:key="key">
                <img :src="data.day.condition.icon" alt="">
                <div class="flex flex-col md:flex-row gap-x-8">
                    <p>Date: {{ data.date }}</p>
                    <p>Avg temp: {{ data.day.avgtemp_c }}°C</p>
                    <p>Condition: {{ data.day.condition.text }}</p>
                </div>
            </div>
        </div>
    </div>
</template>
