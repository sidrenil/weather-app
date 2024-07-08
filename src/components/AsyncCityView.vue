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
      <p class="text-sm mb-12">
        {{
          new Date(weatherData.currentTime).toLocaleDateString("en-us", {
            weekday: "short",
            day: "2-digit",
            month: "long",
          })
        }}
        {{
          new Date(weatherData.currentTime).toLocaleTimeString("en-us", {
            timeStyle: "short",
          })
        }}
      </p>
      <p v-if="weatherData" class="text-4xl mb-2">
        {{ Math.round(weatherData.current.temp) }}&deg;
      </p>

      <p>
        Feels Like
        {{ Math.round(weatherData.current.feels_like) }} &deg;
      </p>
      <p class="capitalize">
        {{ weatherData.current.weather[0].description }}
      </p>
      <img
        class="w-[150px]"
        h-auto
        :src="'http://openweathermap.org/img/wn/03d@2x.png'"
        alt=""
      />
    </div>

    <hr class="border-white border-opacity-10 border w-full" />
    <!---Hourly Weather-->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">Hourly Weather</h2>
        <div class="flex gap-10 overflow-x-scroll">
          <div
            v-for="hourData in weatherData.hourly"
            :key="hourData.dt"
            class="flex flex-col gap-4 items-center"
          >
            <p class="whitespace-nowrap text-md">
              {{
                new Date(hourData.currentTime).toLocaleDateString("en-us", {
                  hour: "numeric",
                })
              }}
            </p>
            <img
              class="w-auto h-[50px] object-cover"
              :src="'https://openweathermap.org/img/wn/01d@2x.png'"
              alt=""
            />
            <p class="text-xl">{{ Math.round(hourData.temp) }}&deg;</p>
          </div>
        </div>
      </div>
    </div>

    <hr class="border-white border-opacity-10 border w-full" />
    <!--Weekly Overview-->

    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">7 Day Forecast</h2>
        <div
          v-for="day in weatherData.daily"
          :key="day.dt"
          class="flex items-center"
        >
          <p class="flex-1">
            {{
              new Date(day.dt * 1000).toLocaleDateString("en-us", {
                weekday: "long",
              })
            }}
          </p>
          <img
            class="w-[150px] h-[150px] object-cover"
            :src="'https://openweathermap.org/img/wn/02d@2x.png'"
            alt=""
          />
          <div class="flex gap-2 flex-1 justify-end">
            <p>H: {{ Math.round(day.temp.max) }}</p>
            <p>L: {{ Math.round(day.temp.min) }}</p>
          </div>
        </div>
      </div>
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
