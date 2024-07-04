<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="mapBoxSearchResults !== null"
      >
        <p class="py-2" v-if="searchError">
          Sorry, something went wrong, please try again.
        </p>
        <p
          class="py-2"
          v-if="
            !searchError &&
            mapBoxSearchResults.length === 0 &&
            searchQuery.trim() !== ''
          "
        >
          No results match your query, try a different term.
        </p>
        <template v-else>
          <li
            v-for="searchResult in mapBoxSearchResults"
            :key="searchResult.id"
            class="py-2 cursor-pointer"
            @click="previewCity(searchResult)"
          >
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
  </main>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";

const router = useRouter();

const previewCity = (searchResult) => {
  console.log(searchResult);
  const [city, state] = searchResult.place_name.split(",");
  console.log(city, state);
  router.push({
    name: "cityView",
    params: { state: state.replaceAll(" ", ""), city: city },
    query: {
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true,
    },
  });
};

const mapboxAPIKey =
  "pk.eyJ1Ijoic2lkcmUiLCJhIjoiY2x5NzE0a3JoMDE4djJrcXdoYmhyYzB0MCJ9.rGIMiWpgE7Ol_u01SqnLgg";
const searchQuery = ref("");
const queryTimeout = ref(null);
const mapBoxSearchResults = ref(null);
const searchError = ref(null);

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value.trim() !== "") {
      const apiUrl = `https://api.mapbox.com/geocoding/v5/mapbox.places/${encodeURIComponent(
        searchQuery.value
      )}.json?access_token=${mapboxAPIKey}`;
      try {
        const result = await axios.get(apiUrl);
        mapBoxSearchResults.value = result.data.features;
        searchError.value = false;
      } catch (error) {
        searchError.value = true;
        mapBoxSearchResults.value = [];
      }
    } else {
      mapBoxSearchResults.value = null;
      searchError.value = null;
    }
  }, 300);
};
</script>

<style lang="scss" scoped></style>
