<script setup>
import axios from 'axios'
import { ref, onMounted, watch } from 'vue'

const daysOfWeek = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];

const items = ref()
const forecast = ref()
const apiKey = import.meta.env.VITE_WEATHER_API_KEY

const baseURL = "https://api.weatherapi.com/v1/"

const location = ref('Chiang Mai')

function fetchWeather() {
    let params = {
        key: apiKey,
        q: location.value,
    }
    axios.get(baseURL + 'current.json', { params }).then((response) => {
        items.value = response.data
        location.value = '#' + response.data.location.name + ', ' + response.data.location.country
        updateInput(location.value)
    }).catch((err) => {
        console.log(err);
    })
}

function fetchForecast() {
    let params = {
        key: apiKey,
        q: location.value,
        days: 1
    }
    axios.get(baseURL + 'forecast.json', { params }).then((response) => {
        forecast.value = response.data
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
            fetchForecast()
        } else {
            location.value = 'Chiang Mai'
        }
    }
})

onMounted(fetchWeather)
onMounted(fetchForecast)
</script>

<template>
    <div v-if="!items" class="flex flex-col max-w-4xl mx-auto w-full">
        <div class="p-8 flex justify-center">
            <p class="text-neutral-600 font-medium text-xl">I'm trying to get the data for you please wait. ;w;</p>
        </div>
    </div>
    <div v-if="items" class="max-w-4xl mx-auto">
        <div class="relative flex">
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"
                class="w-5 h-5 text-neutral-500 top-1/2 transform -translate-y-1/2 left-5 absolute">
                <path stroke-linecap="round" stroke-linejoin="round"
                    d="M21 21l-5.197-5.197m0 0A7.5 7.5 0 105.196 5.196a7.5 7.5 0 0010.607 10.607z" />
            </svg>
            <input id="inputLocation" v-model.lazy="location"
                class="py-1 bg-neutral-800 flex-grow rounded-md text-neutral-500 ml-4 mr-4 pl-7 font-medium text-xl"
                type="text">
        </div>
        <div class="grid grid-cols-2 md:grid-cols-3 gap-2 p-4 text-slate-100">
            <div class="p-4 col-span-2 md:col-span-1 border border-neutral-600 rounded-md flex flex-col justify-center">
                <div class="flex flex-row gap-x-4 items-center justify-center">
                    <img class="object-contain" :src="items.current.condition.icon" alt="" width="64" height="64">
                    <div class="flex flex-col">
                        <p class="font-medium text-gray-50 text-5xl">{{ items.current.temp_c }}°C</p>
                        <p class="text-neutral-600 text-lg font-medium self-start">{{ daysOfWeek[new
                            Date(items.current.last_updated).getDay()] }}</p>
                    </div>
                </div>
            </div>
            <div class="p-4 border border-neutral-600 rounded-md flex flex-col items-center justify-center">
                <div>
                    <div class="flex flex-col md:flex-row transition-all duration-200">
                        <p class="font-medium text-gray-50 text-4xl mr-2">Feels
                        </p>
                        <p v-if="items.current.feelslike_c >= 30" class="font-medium text-4xl text-orange-500">{{
                            items.current.feelslike_c.toFixed(0)
                        }}°C</p>
                        <p v-else-if="items.current.feelslike_c >= 25" class="font-medium text-4xl text-orange-400">{{
                            items.current.feelslike_c.toFixed(0)
                        }}°C</p>
                        <p v-else-if="items.current.feelslike_c >= 22" class="font-medium text-4xl text-yellow-400">{{
                            items.current.feelslike_c.toFixed(0)
                        }}°C</p>
                        <p v-else-if="items.current.feelslike_c >= 18" class="font-medium text-4xl text-green-400">{{
                            items.current.feelslike_c.toFixed(0)
                        }}°C</p>
                        <p v-else-if="items.current.feelslike_c >= 14" class="font-medium text-4xl text-green-300">{{
                            items.current.feelslike_c.toFixed(0)
                        }}°C</p>
                        <p v-else-if="items.current.feelslike_c >= 10" class="font-medium text-4xl text-sky-300">{{
                            items.current.feelslike_c.toFixed(0)
                        }}°C</p>
                        <p v-else-if="items.current.feelslike_c >= 7" class="font-medium text-4xl text-blue-400">{{
                            items.current.feelslike_c.toFixed(0)
                        }}°C</p>
                        <p v-else class="font-medium text-4xl text-blue-500">{{
                            items.current.feelslike_c.toFixed(0)
                        }}°C</p>
                    </div>
                </div>
            </div>
            <div class="p-4 border border-neutral-600 rounded-md flex flex-col justify-center">
                <div class="flex flex-row gap-x-4 items-center justify-center">
                    <div class="flex flex-col">
                        <p class="font-medium text-gray-50 text-5xl">{{ items.current.humidity }}%</p>
                        <p class="text-neutral-600 text-lg font-medium self-start">Humidity</p>
                    </div>
                </div>
            </div>
            <div v-if="forecast" class="p-2 flex flex-col border border-neutral-600 rounded-md col-span-2">
                <p class="mt-2 px-2 text-base font-medium text-sky-400">Forecast hourly</p>
                <div class="mt-2 flex overflow-hidden hover:overflow-auto">
                    <div class="flex flex-col shrink-0 items-center text-xl font-medium"
                        v-for="(item, key) in forecast.forecast.forecastday[0].hour" v-bind:key="key">
                        <p class="text-xl">{{ key }}</p>
                        <img class="m-2 w-12 h-12 object-cover" :src="item.condition.icon" alt="">
                        <p>{{ (item.temp_c).toFixed(0) }}°</p>
                    </div>
                </div>
            </div>
            <div class="p-4 col-span-1 border border-neutral-600 rounded-md flex flex-col justify-center">
                <div class="flex items-center justify-center">
                    <p class="font-semibold text-neutral-50 text-6xl mr-2">{{ items.current.wind_kph }}</p>
                    <div class="flex flex-col">
                        <p class="font-medium text-neutral-600 text-xl">km/h</p>
                        <p class="text-green-600 text-base font-medium self-start">wind speed</p>
                    </div>
                </div>
            </div>
            <div class="p-4 col-span-1 border border-neutral-600 rounded-md flex flex-col justify-center">
                <div class="flex items-center justify-center">
                    <div class="flex flex-col">
                        <p class="font-semibold text-neutral-400 text-xl">Wind blowing</p>
                        <div class="flex">
                            <p class="font-semibold text-5xl text-neutral-50 mr-2">{{ items.current.wind_dir }}</p>
                            <div class="flex flex-col">
                                <p class="text-neutral-600 text-base font-medium">at a degree
                                </p>
                                <p class="text-neutral-600 text-base font-medium">{{ items.current.wind_degree }}°
                                </p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div
                class="p-4 border border-neutral-600 rounded-md transition-all duration-200 flex flex-col items-center justify-center">
                <p class="mt-2 px-2 text-base font-medium text-sky-400">Visibility</p>
                <p>{{ items.current.vis_km }} KM</p>
                <p class="mt-2 px-2 text-base font-medium text-sky-400">Pressure</p>
                <p>{{ items.current.pressure_mb }} mbar</p>
                <p class="mt-2 px-2 text-base font-medium text-sky-400">Precipitation</p>
                <p class="text-blue-400 font-medium">{{ items.current.precip_mm }} MM</p>
            </div>

        </div>
    </div>
</template>
