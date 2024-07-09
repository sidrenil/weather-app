<template>
  <div>
    <div v-for="city in savedCities" :key="city.id">
      <CityCard :city="city" @click="goToCityView(city)" />
    </div>
    <p v-if="savedCities.length === 0">
      No locations added. To start tracking a location, search in the field
      above.
    </p>
  </div>
</template>

<script setup>
import axios from "axios";
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
import CityCard from "./CityCard.vue";

const savedCities = ref([]);

const getCities = async () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(localStorage.getItem("savedCities"));
    const requests = savedCities.value.map((city) => {
      if (city.coords && city.coords.lat && city.coords.lng) {
        return axios.get(
          `https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=3659b7a8b5e7572358698ad695eddfb9&units=metric`
        );
      } else {
        return Promise.resolve(null);
      }
    });

    try {
      const weatherData = await Promise.all(requests);
      weatherData.forEach((value, index) => {
        if (value) {
          savedCities.value[index].weather = value.data;
        }
      });
    } catch (error) {
      console.error("Error fetching weather data:", error);
    }
  }
};

onMounted(async () => {
  await getCities();
});

const router = useRouter();
const goToCityView = (city) => {
  router.push({
    name: "cityView",
    params: { state: city.state, city: city.city },
    query: { lat: city.coords.lat, lng: city.coords.lng },
  });
};
</script>
