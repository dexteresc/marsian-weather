<template>
  <div class="Weather">
    <div class="current-weather">
      <div class="main">
        <div class="city">{{ apiResponse.name }}</div>
        <div class="time">{{ apiResponse.sys.country }} {{ getTime() }}</div>
        <div class="degrees">{{ apiResponse.main.temp }}°</div>
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
          <span class="high"> {{ apiResponse.main.temp_max }}°</span>
          /
          <span class="low"> {{ apiResponse.main.temp_min }}°</span>
        </div>
      </div>
    </div>
    <div class="wind">
      <div class="wind-icon">
        <img
          v-bind:src="
            require('@/assets/icons/line/all/wind-beaufort-' +
              Math.round(apiResponse.wind.speed) +
              '.svg')
          "
          alt=""
          srcset=""
        />
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Weather",
  props: {
    apiResponse2: Object,
  },
  data() {
    return {
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
      unit: "C",
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
  },
};
</script>

<style lang="scss" scoped>
@import "../styles/_globals.scss";
.Weather {
  width: 80%;
  margin: 2em auto;
  .current-weather {
    & > * {
      padding: 2em;
    }
    background-color: #fafafa;
    justify-content: space-between;
    overflow: hidden;
    border-radius: 25px;
    display: flex;
    .main {
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
</style>