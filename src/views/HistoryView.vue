<script setup>
import axios from 'axios'
import { ref, onMounted, watch } from 'vue'

const items = ref()
const apiKey = import.meta.env.VITE_WEATHER_API_KEY

const baseURL = "http://api.weatherapi.com/v1/"

const location = ref('Chiang Mai')

function fetchHistoryWeather() {
    const start_dt = new Date()
    const end_dt = new Date()
    end_dt.setDate(end_dt.getDate() - 5)
    const params = {
        key: apiKey,
        q: location.value,
        dt: end_dt,
        end_dt: start_dt
    }
    axios.get(baseURL + "history.json", { params }).then((response) => {
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
            fetchHistoryWeather()
        } else {
            location.value = 'Chiang Mai'
        }
    }
})

onMounted(fetchHistoryWeather)

</script>

<template>
    <div class="text-white flex flex-col items-center justify-center">
        <div v-if="items" class="flex flex-col max-w-3xl w-full">
            <p class="p-4 font-semibold self-center text-4xl">Last weather in 5 days</p>
            <input id="inputLocation" v-model.lazy="location"
                class="bg-neutral-900 focus:bg-neutral-100 focus:text-gray-950 w-auto px-4 font-medium text-xl" type="text">
            <p class="text-gray-400 px-4">Weather history</p>
            <div class="m-4 p-4 flex items-center border rounded-md gap-x-8 hover:bg-neutral-700"
                v-for="(data, key) in items.forecast.forecastday.slice().reverse()" v-bind:key="key">
                <img :src="data.day.condition.icon" alt="">
                <div class="flex flex-col md:flex-row gap-x-8">
                    <p>Date: {{ data.date }}</p>
                    <p>Avg temp: {{ data.day.avgtemp_c }} °C</p>
                    <p>Condition: {{ data.day.condition.text }}</p>
                </div>
            </div>
        </div>
    </div>
</template>
