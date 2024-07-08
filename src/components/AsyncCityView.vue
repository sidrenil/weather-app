<template>
  <div class="flex flex-col flex-1 items-center">
    <!--Banner-->
    <div
      v-if="route.query.preview"
      class="text-white p-4 bg-weather-secondary w-full text-center"
    >
      <p>
        You are currently previewing this city, click the "+" icon to start
        tracking this city.
      </p>
    </div>
    <!--Weather Overview-->
    <div class="flex flex-col items-center text-white py-12">
      <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
      <p v-if="weatherData" class="text-4xl mb-2">
        {{ Math.round(weatherData.current.temp) }}&deg;
      </p>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { ref, onMounted } from "vue";
import { useRoute } from "vue-router";

const route = useRoute();
const weatherData = ref(null);

const getWeatherData = async (id) => {
  try {
    const response = await axios.get(
      `https://api.openweathermap.org/data/3.0/onecall?lat=${route.query.lat}&lon=${route.query.lng}&exclude={part}&appid=3659b7a8b5e7572358698ad695eddfb9&units=metric`
    );
    console.log("baty", "response.data");

    const data = response.data;
    const localOffset = new Date().getTimezoneOffset() * 60000;
    const utc = data.current.dt * 1000 + localOffset;
    data.currentTime = utc + 1000 * data.timezone_offset;

    data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset;
      hour.currentTime = utc + 1000 * data.timezone_offset;
    });

    weatherData.value = data; // weatherData'yı güncelleyin
  } catch (err) {
    console.error("Hava durumu verisi alınırken hata oluştu:", err);
  }
};

onMounted(async () => {
  await getWeatherData();
});
</script>
