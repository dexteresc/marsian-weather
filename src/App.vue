<template>
  <Navbar
    v-model="searchText"
    v-on:tempUnit="changeUnit"
    :unit="unit"
    v-on:searchClick="getWeather"
  />
  <Weather :apiResponse="apiResponse" :unit="unit" />
  <div class="search-error" v-if="apiResponse && apiResponse.cod != 200">
    City not found.
  </div>
  <div v-if="!apiResponse && searchText" class="search-error">
    "{{ searchText }}" not found
  </div>
</template>

<script>
import { ref } from "vue";
import Navbar from "./components/Navbar.vue";
import Weather from "./components/Weather.vue";
import apiKey from "./apiKey.json";

export default {
  // v-if="apiResponse && apiResponse.cod == 200"
  name: "App",
  components: {
    Navbar,
    Weather,
  },
  data() {
    return {
      unit: false,
      apiResponse: "",
    };
  },
  setup() {
    const searchText = ref("");
    return { searchText };
  },
  methods: {
    changeUnit(tempUnit) {
      this.unit = tempUnit;
      if (this.searchText) {
        this.getWeather();
      }
    },
    getWeather() {
      let searchQuery = this.searchText.trim();
      console.log(searchQuery);
      if (searchQuery == "" || /\d/.test(searchQuery)) {
        return 0;
      } else {
        let sUnit;
        if (this.unit) {
          sUnit = "imperial";
        } else {
          sUnit = "metric";
        }
        fetch(
          "https://api.openweathermap.org/data/2.5/weather?q=" +
            searchQuery +
            "&appid=" +
            apiKey.apiKey +
            "&units=" +
            sUnit
        ).then((res) =>
          res
            .json()
            .then((data) => {
              console.log(data);
              this.apiResponse = data;
            })
            .catch((error) => {
              console.log(error);
            })
        );
      }
    },
  },
};
</script>

<style lang="scss">
@import "./styles/_globals.scss";

@font-face {
  font-family: "Avenir";
  src: url("./assets/fonts/Avenir/Avenir Regular/Avenir Regular.ttf")
    format("truetype");
  font-weight: normal;
}
@font-face {
  font-family: "Avenir";
  src: url("./assets/fonts/Avenir/Avenir Light/Avenir Light.ttf")
    format("truetype");
  font-weight: 300;
}
@font-face {
  font-family: "Avenir";
  src: url("./assets/fonts/Avenir/Avenir Heavy/Avenir Heavy.ttf")
    format("truetype");
  font-weight: bold;
}
@font-face {
  font-family: "Avenir";
  src: url("./assets/fonts/Avenir/Avenir Black/Avenir Black.ttf")
    format("truetype");
  font-weight: 800;
}
head,
body {
  margin: 0;
  padding: 0;
}
body {
  background-color: $divider-color;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
}
</style>
