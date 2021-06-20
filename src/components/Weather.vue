<template>
  <div class="Weather">
    <div class="current-weather">
      <div class="current-temperature">
        <div class="main">
          <div class="city">{{ apiResponse.name }}</div>
          <div class="time">{{ apiResponse.sys.country }} {{ getTime() }}</div>
          <div class="degrees">{{ Math.round(apiResponse.main.temp) }}°</div>
          <div class="description">{{ apiResponse.weather[0].main }}</div>
        </div>
        <div class="temperature">
          <div class="weather-icon">
            <img
              v-bind:src="
                require('@/assets/icons/line/openweathermap/' +
                  apiResponse.weather[0].icon +
                  '.svg')
              "
              alt=""
            />
          </div>
          <div class="highlow">
            <span class="high">
              {{ Math.round(apiResponse.main.temp_max) }}°</span
            >
            /
            <span class="low">
              {{ Math.round(apiResponse.main.temp_min) }}°</span
            >
          </div>
        </div>
      </div>
      <div class="wind">
        <h1>Wind</h1>
        <div class="wind-icon">
          <img
            v-bind:src="
              require('@/assets/icons/line/all/wind-beaufort-' +
                beaufortScale(apiResponse.wind.speed) +
                '.svg')
            "
            alt=""
            srcset=""
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Weather",
  props: {
    apiResponse2: Object,
    unit: Boolean,
  },
  data() {
    return {
      // Testing
      apiResponse: {
        coord: {
          lon: -122.08,
          lat: 37.39,
        },
        weather: [
          {
            id: 800,
            main: "Clear",
            description: "clear sky",
            icon: "01d",
          },
        ],
        base: "stations",
        main: {
          temp: 282.55,
          feels_like: 281.86,
          temp_min: 280.37,
          temp_max: 284.26,
          pressure: 1023,
          humidity: 100,
        },
        visibility: 16093,
        wind: {
          speed: 1.5,
          deg: 350,
        },
        clouds: {
          all: 1,
        },
        dt: 1560350645,
        sys: {
          type: 1,
          id: 5122,
          message: 0.0139,
          country: "US",
          sunrise: 1560343627,
          sunset: 1560396563,
        },
        timezone: -25200,
        id: 420006353,
        name: "Mountain View",
        cod: 200,
      },
    };
  },
  methods: {
    getTime() {
      let time = new Date(this.apiResponse.dt * 1000);
      let hour = time.getHours();
      let minute = time.getMinutes();
      let s = hour + ":" + minute;
      return s;
    },
    beaufortScale(value) {
      if (this.unit) {
        // Imperial
        if (value < 1) return 0;
        if (value < 4) return 1;
        if (value < 8) return 2;
        if (value < 13) return 3;
        if (value < 19) return 4;
        if (value < 25) return 5;
        if (value < 32) return 6;
        if (value < 39) return 7;
        if (value < 47) return 8;
        if (value < 55) return 9;
        if (value < 64) return 10;
        if (value < 73) return 11;
        if (value >= 73) return 12;
      } else {
        // Metric
        if (value < 0.5) return 0;
        if (value < 1.6) return 1;
        if (value < 3.4) return 2;
        if (value < 5.6) return 3;
        if (value < 8) return 4;
        if (value < 10.8) return 5;
        if (value < 13.9) return 6;
        if (value < 17.2) return 7;
        if (value < 20.8) return 8;
        if (value < 24.5) return 9;
        if (value < 28.5) return 10;
        if (value < 32.7) return 11;
        if (value >= 32.7) return 12;
      }
    },
  },
};
</script>

<style lang="scss" scoped>
@import "../styles/_globals.scss";
.Weather {
  width: 80%;
  margin: 2em auto;
  background-color: #fafafa;
  border-radius: 25px;
  overflow: hidden;
  .current-weather {
    .current-temperature {
      justify-content: space-between;
      display: flex;
      & > * {
        padding: 2em;
      }
      display: flex;
      .main {
        border-radius: 25px;
        background-color: $primary-color;
        color: $primary-color-text;
        font-weight: bolder;
        display: flex;
        flex-direction: column;
        .city {
          font-size: 1.5em;
        }
        .country {
          font-size: 0.8rem;
        }
        .time {
          font-weight: lighter;
        }
      }
      .weather-icon img {
        width: 10em;
      }
      .temperature {
        height: 100%;
        text-align: left;
        .highlow {
          text-align: left;
          .high {
            color: red;
          }
          .low {
            color: blue;
          }
        }
      }
      .degrees {
        font-size: 4em;
        font-weight: 600;
      }
    }
  }
}
</style>