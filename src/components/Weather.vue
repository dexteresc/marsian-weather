<template>
  <div class="Weather">
    <div class="current-weather">
      <div class="main">
        <div class="city">{{ apiResponse.name }}</div>
        <div class="time">{{ apiResponse.sys.country }} {{ getTime() }}</div>
        <div class="weather-icon">
          <img
            v-bind:src="
              require('@/assets/icons/line/openweathermap/' +
                apiResponse.weather[0].icon +
                '.svg')
            "
            alt=""
          />
          <div class="description">{{ apiResponse.weather[0].main }}</div>
        </div>
      </div>
      <div class="temperature">
        <div class="degrees">31 °</div>
        <div class="highlow">
          <span class="high"> {{ apiResponse.main.temp_max }} °</span>
          /
          <span class="low"> {{ apiResponse.main.temp_min }} °</span>
        </div>
      </div>
    </div>
    <div class="wind">
      <div class="wind-icon">
        <img v-bind:src="require('@/assets/icons/line/all/wind-beaufort-' + Math.round(apiResponse.wind.speed) + '.svg')" alt="" srcset="">
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
  created() {
    console.log(
      "../assets/icons/line/openweathermap/" +
        this.apiResponse.weather[0].icon +
        ".svg"
    );
  },
};
</script>

<style lang="scss" scoped>
$icon-size: 5em;
.Weather {
  .current-weather {
    background: rgb(2, 0, 36);
    background: linear-gradient(
      90deg,
      rgba(2, 0, 36, 1) 0%,
      rgba(175, 151, 203, 1) 0%,
      rgba(0, 212, 255, 1) 100%
    );
    padding: 2em;
    justify-content: space-evenly;
    display: flex;
    .main {
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
      .weather-icon img {
        width: $icon-size;
      }
    }
    .temperature {
      height: 100%;
      text-align: left;
      .degrees {
        font-size: 5em;
        font-weight: 300;
      }
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
  }
}
</style>