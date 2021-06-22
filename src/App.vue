<template>
  <div class="app">
    <div class="sidebar">
      <header>
        <div class="search-bar">
          <span class="material-icons-outlined"> search </span>
          <input
              v-model="searchText"
              placeholder="Search for places..."
              type="text"
          />
          <v-button v-if="!myLocation.loading"
                    :class="{'error': myLocation['my-location-error'], 'active': myLocation['my-location-active']}"
                    class="accent rounded"
                    v-on:click="getCurrentLocation"
          >
          <span
              class="material-icons-outlined"
              :title="myLocation['my-location-error'] ? 'Something' : null"
          >
                {{ locationIcon }}
          </span>
          </v-button>
          <div v-if="myLocation.loading">Loading</div>
        </div>
      </header>
      <section class="current-weather">
        <div class="current-weather-icon">
          <img
              alt=""
              v-bind:src="
              require('@/assets/icons/line/openweathermap/' +
                apiResponse.weather[0].icon +
                '.svg')
            "
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
        <LocationWidget :location="apiResponse.name"/>
      </footer>
    </div>

    <main>
      <nav>
        <ul>
          <li :class="{ 'tab-active': !tab }">Today</li>
          <li :class="{ 'tab-active': tab }">Week</li>
        </ul>
        <div class="units">
          <v-button :class="{ 'primary': !unitActive }" class="unit rounded">°C</v-button>
          <v-button :class="{ 'primary': unitActive }" class="unit rounded">°F</v-button>
        </div>
      </nav>
      <div class="weather-highlights">
        <div class="container">UV Index</div>
        <div class="container">Wind Status</div>
        <div class="container">Sunrise & Sunset</div>
        <div class="container">Humidity</div>
        <div class="container">Visibility</div>
        <div class="container">Air Quality</div>
      </div>
    </main>
  </div>
</template>

<script>
import apiKey from "./apiKey.json";
import CurrentWeatherProperty from "./components/current-weather-property.vue";
import LocationWidget from "./components/location-widget.vue";
import Button from "./components/button"

export default {
  name: "App",
  components: {
    CurrentWeatherProperty,
    LocationWidget,
    'v-button': Button,
  },
  data() {
    return {
      tab: false,
      myLocation: {
        coords: "",
        'my-location-active': false,
        "my-location-error": false,
        loading: false,
      },
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
    return {dayOfWeek};
  },
  mounted() {
    this.date = new Date(this.apiResponse.dt * 1000);

  },
  methods: {
    getWeatherByCoords(pos) {
      const crd = pos.coords;
      console.log('Your current position is:');
      console.log(`Latitude : ${crd.latitude}`);
      console.log(`Longitude: ${crd.longitude}`);
      console.log(`More or less ${crd.accuracy} meters.`);
    },
    getWeatherByQuery() {
      let query;
      let sUnit;
      query = this.searchText.trim();
      if (query === "" || /\d/.test(query)) {
        return 0;
      } else {
        if (this.unit) {
          sUnit = "imperial";
        } else {
          sUnit = "metric";
        }
        fetch(
            "https://api.openweathermap.org/data/2.5/weather?q=" +
            query +
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
    getCurrentLocation() {
      console.log("this ran")

      function report(state) {
        console.log("Permission " + state)
      }

      this.myLocation.loading = true;

      navigator.permissions.query({
        name: 'geolocation',
      }).then((result) => {
        if (result.state === 'granted') {
          report(result.state)
          this.myLocation["my-location-active"] = true;
        } else if (result.state === 'prompt') {
          console.log(this.myLocation.loading)
          report(result.state);
          navigator.geolocation.getCurrentPosition(this.getWeatherByCoords)

        } else if (result.state === 'denied') {
          report(result.state);
          this.myLocation["my-location-error"] = true
        }
        result.onchange = () => {
          report(result.state);
          if (result.state === 'denied') {
            this.myLocation["my-location-error"] = true;
          }
        }
      }).finally(() => {
        this.myLocation.loading = false
      })

    },

  },
  computed: {
    locationIcon() {
      if (this.myLocation["my-location-error"]) {
        return "location_off"
      } else if (this.myLocation["my-location-granted"]) {
        return "location_on";
      } else {
        return "location_on"
      }
    },
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
    unitActive() {
      return this.unit.temp !== "C";
    },
  },
};
</script>

<style lang="scss">
@import "./styles/_globals.scss";

@font-face {
  font-family: "Avenir";
  src: url("./assets/fonts/Avenir/Avenir Regular/Avenir Regular.ttf") format("truetype");
  font-weight: normal;
}

@font-face {
  font-family: "Avenir";
  src: url("./assets/fonts/Avenir/Avenir Light/Avenir Light.ttf") format("truetype");
  font-weight: 300;
}

@font-face {
  font-family: "Avenir";
  src: url("./assets/fonts/Avenir/Avenir Heavy/Avenir Heavy.ttf") format("truetype");
  font-weight: bold;
}

@font-face {
  font-family: "Avenir";
  src: url("./assets/fonts/Avenir/Avenir Black/Avenir Black.ttf") format("truetype");
  font-weight: 800;
}

head,
body {
  margin: 0;
}

body {
  background-color: $accent-color;
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
  background-color: $primary-color;
  width: 25%;

  header {
    flex: 0 1 auto;

    .search-bar {
      display: flex;
      width: 100%;
      align-items: center;

      input {
        flex: 1 1 auto;
        min-width: 0;
        background-color: transparent;
        border: 0;
        outline: 0;
        font-size: 1em;
        padding: 0 0.5em;
      }

      &.my-location-active {
        color: #3f50b5;
      }
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


main {
  flex: 1 1 auto;

  nav {
    width: 100%;
    display: flex;
    justify-content: space-between;
    align-items: center;
    box-sizing: border-box;
    padding: 0 1em;

    ul {
      list-style: none;
      padding: 0;
      margin: 0;
      display: flex;
      align-items: center;

      li {
        font-weight: bold;
        margin-right: 1em;
        color: $accent-text-color;
      }

      li.tab-active {
        border-bottom: #000 solid 2px;
        color: $primary-text-color;
      }
    }

    .units {
      display: flex;
      align-items: center;

      .unit {
        $unit-size: 40px;
        font-size: 1em;
        margin-left: 1em;;
        width: $unit-size;
        height: $unit-size;
        border-radius: 50%;
      }
    }
  }
}

.container {
  padding: 1em;
  margin: 1em;
  background-color: $primary-color;
  border-radius: 15px;
}

.divider {
  margin: 1em auto;
  display: block;
  width: 100%;
  height: 1px;
  background-color: $accent-color;
}

</style>
