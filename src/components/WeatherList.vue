<script setup>
import axios from 'axios'
import { ref, onMounted, watch, defineProps } from 'vue'

const props = defineProps(['type'])

const daysOfWeek = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];

const items = ref()
const apiKey = import.meta.env.VITE_WEATHER_API_KEY

const baseURL = "http://api.weatherapi.com/v1/"

const location = ref('Chiang Mai')

function fetchWeather() {
    let params = {
        key: apiKey,
        q: location.value,
        days: 5,
    }

    let endpoints = '';
    const start_dt = new Date()
    const end_dt = new Date()

    switch (props.type) {
        case 'forecast':
            endpoints = 'forecast.json';
            break;
        case 'history':
            endpoints = 'history.json';
            end_dt.setDate(end_dt.getDate() - 5)
            params = {
                key: apiKey,
                q: location.value,
                dt: end_dt,
                end_dt: start_dt
            }
            break;
        default:
            endpoints = 'forecast.json'
            break;
    }

    axios.get(baseURL + endpoints, { params }).then((response) => {
        //console.log(response.data)
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
            fetchWeather()
        } else {
            location.value = 'Chiang Mai'
        }
    }
})

onMounted(fetchWeather)
</script>

<template>
    <div v-if="items" class="flex flex-col max-w-4xl w-full">
        <p v-if="type === 'history'" class="p-4 font-semibold self-center text-4xl">Last weather in 5 days</p>
        <p v-if="type === 'forecast'" class="p-4 font-semibold self-center text-4xl">{{
            items.forecast.forecastday.length }}-days forecast</p>
        <div class="relative flex">
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"
                class="w-5 h-5 text-sky-600 top-1/2 transform -translate-y-1/2 left-5 absolute">
                <path stroke-linecap="round" stroke-linejoin="round"
                    d="M21 21l-5.197-5.197m0 0A7.5 7.5 0 105.196 5.196a7.5 7.5 0 0010.607 10.607z" />
            </svg>
            <input id="inputLocation" v-model.lazy="location"
                class="bg-gray-800 flex-grow rounded-md text-sky-500 ml-4 mr-4 pl-7 font-medium text-xl" type="text">
        </div>
        <p v-if="type === 'forecast'" class="text-gray-400 px-4">Weather forecast</p>
        <p v-if="type === 'history'" class="text-gray-400 px-4">Weather history</p>
        <div class="group m-4 px-4 py-4 grid grid-cols-3 md:h-32 items-center transition-all duration-200 border border-gray-600 rounded-md gap-x-4 hover:border-gray-800 hover:bg-gray-800"
            v-for="(data, key) in items.forecast.forecastday.slice().reverse()" v-bind:key="key">
            <div class="flex flex-col items-center">
                <img :src="data.day.condition.icon" alt="" width="64" height="64">
                <p class="font-medium text-xl">{{ daysOfWeek[new Date(data.date).getDay()] }}</p>
            </div>
            <div class="grid grid-cols-1">
                <p class="md:hidden font-base text-xl">{{ (data.day.avgtemp_c).toFixed(0) }}°C</p>
                <p class="md:hidden font-base text-xl">{{ data.day.condition.text }}</p>
                <p class="hidden md:block">Avg temp: {{ (data.day.avgtemp_c).toFixed(0) }}°C</p>
                <p class="hidden md:block">Condition: {{ data.day.condition.text }}</p>
            </div>
            <div class="md:flex flex-col font-base">
                <p>Humidity: {{ data.day.avghumidity }}%</p>
                <p>UV Index: {{ data.day.uv }}</p>
                <p>Wind: {{ data.day.maxwind_kph }}</p>
            </div>
        </div>
    </div>
</template>