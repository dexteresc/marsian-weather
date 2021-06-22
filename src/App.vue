<template>
  <div class="app">
    <div class="sidebar">
      <header>
        <div class="search-bar">
          <span class="material-icons-outlined"> search </span>
          <input
            type="text"
            v-model="searchText"
            placeholder="Search for places..."
          />
          <div class="my-location">X</div>
        </div>
      </header>
      <section class="current-weather">
        <div class="current-weather-icon">
          <img
            v-bind:src="
              require('@/assets/icons/line/openweathermap/' +
                apiResponse.weather[0].icon +
                '.svg')
            "
            alt=""
          />
        </div>
        <div class="current-temperature">
          {{ Math.round(apiResponse.main.temp) }}<sup>°{{ unit.temp }}</sup>
        </div>
        <div class="current-dt">
          <span> {{ dayOfWeek[date.getDay()] }}, </span>
          {{ getTime }}
        </div>
        <div class="divider"></div>
        <div class="current-weather-properties">
          <CurrentWeatherProperty
            v-for="(o, key) in getCurrentWeatherProperties"
            :key="key"
            v-bind="o"
          />
        </div>
      </section>
      <footer>
        <LocationWidget :location="apiResponse.name" />
      </footer>
    </div>

    <main>
      <nav>
        <ul>
          <li>Today</li>
          <li>Week</li>
        </ul>
      </nav>
      <div class="weather">
        <div class="container">temperature</div>
        <div class="container">Wind</div>
        <div class="container">{{ searchText }}</div>
      </div>
    </main>
  </div>
</template>

<script>
import apiKey from "./apiKey.json";
import CurrentWeatherProperty from "./components/current-weather-property.vue";
import LocationWidget from "./components/location-widget.vue";
export default {
  name: "App",
  components: {
    CurrentWeatherProperty,
    LocationWidget,
  },
  data() {
    return {
      unit: {
        temp: "C",
        rain: "mm",
        wind: "m/s",
      },
      searchText: "",
      date: new Date(),
      apiResponse: {
        coord: {
          lon: -75.6826,
          lat: 44.5834,
        },
        weather: [
          {
            id: 500,
            main: "Rain",
            description: "light rain",
            icon: "10d",
          },
        ],
        base: "stations",
        main: {
          temp: 20.61,
          feels_like: 21.13,
          temp_min: 20.08,
          temp_max: 22.54,
          pressure: 999,
          humidity: 92,
        },
        visibility: 10000,
        wind: {
          speed: 2.68,
          deg: 178,
          gust: 4.47,
        },
        rain: {
          "1h": 0.15,
        },
        clouds: {
          all: 100,
        },
        dt: 1624280937,
        sys: {
          type: 2,
          id: 2003785,
          country: "CA",
          sunrise: 1624267062,
          sunset: 1624323083,
        },
        timezone: -14400,
        id: 5909294,
        name: "Brockeville",
        cod: 200,
      },
    };
  },
  setup() {
    const dayOfWeek = [
      "Sunday",
      "Monday",
      "Tuesday",
      "Wednesday",
      "Thursday",
      "Friday",
      "Saturday",
    ];
    return { dayOfWeek };
  },
  mounted() {
    this.date = new Date(this.apiResponse.dt * 1000);
  },
  methods: {
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
  computed: {
    getTime() {
      return this.date.toLocaleTimeString(navigator.language, {
        hour: "2-digit",
        minute: "2-digit",
      });
    },
    getCurrentWeatherProperties() {
      if (!this.apiResponse) return null;
      let arr = [];
      if (this.apiResponse.snow) {
        let snow = {
          icon: "snowflake.svg",
          value: "Snow — " + this.apiResponse.snow["1h"],
          unit: this.unit.rain,
        };
        arr.push(snow);
      } else if (this.apiResponse.rain) {
        let rain = {
          icon: "raindrops.svg",
          value: "Rain — " + this.apiResponse.rain["1h"],
          unit: this.unit.rain,
        };
        arr.push(rain);
      }
      if (this.apiResponse.clouds.all > 0) {
        let clouds = {
          icon: "cloudy.svg",
          value: "Cloudiness — " + this.apiResponse.clouds.all,
          unit: "%",
        };
        arr.push(clouds);
      }
      return arr;
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
}
body {
  background-color: $divider-color;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
}
.app {
  display: flex;
  min-height: 100vh;
  height: auto;
  & > * {
    padding: 2rem;
  }
}
.sidebar {
  display: flex;
  flex-flow: column;
  box-sizing: border-box;
  background-color: #fafafa;
  width: 25%;
  header {
    flex: 0 1 auto;

    .search-bar {
      display: flex;
      width: 100%;
      input {
        flex: 1 1 auto;
        min-width: 0;
        background-color: transparent;
        border: 0;
        outline: 0;
        font-size: 1em;
      }
    }
  }
  .current-weather {
    flex: 1 1 auto;
    display: flex;
    flex-flow: column;
    .current-weather-icon {
      width: 100%;
    }
    .current-temperature {
      font-size: 5em;
      font-weight: 300;
      sup {
        font-size: 0.5em;
        font-weight: normal;
      }
    }
    .current-weather-properties {
      margin-bottom: 1em;
    }
  }
  footer {
    flex: 0 1 80px;
    width: 100%;
  }
}
main {
  flex: 1 1 auto;
}
.container {
  padding: 1em;
  margin: 1em;
  background-color: #fafafa;
  border-radius: 15px;
}
.divider {
  margin: 1em auto;
  display: block;
  width: 100%;
  height: 1px;
  background-color: $divider-color;
}
</style>
