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
              @keyup.enter="forecast(searchText, $event)"
          />
          <v-button
              id="geolocation"
              :class="{error: myLocation['my-location-error'], active: myLocation['my-location-active']}"
              class="accent rounded"
              @click="currentLocation"
          >
          <span
              v-if="!myLocation.loading"
              :title="myLocation['my-location-error'] ? 'Location disabled for website in chrome' : null"
              class="material-icons-outlined"
          >
                {{ locationIcon }}
          </span>
            <svg v-else id="L9" enable-background="new 0 0 0 0"
                 viewBox="0 0 100 100" x="0px"
                 xml:space="preserve" xmlns="http://www.w3.org/2000/svg" y="0px">
    <path d="M73,50c0-12.7-10.3-23-23-23S27,37.3,27,50 M30.9,50c0-10.5,8.5-19.1,19.1-19.1S69.1,39.5,69.1,50"
          fill="#000">
      <animateTransform
          attributeName="transform"
          attributeType="XML"
          dur="1s"
          from="0 50 50"
          repeatCount="indefinite"
          to="360 50 50"
          type="rotate"/>
  </path>
</svg>
          </v-button>
        </div>
      </header>
      <section class="current-weather">
        <div class="current-weather-icon">
          <img
              :src="require('@/assets/icons/line/all/' + ( api.forecast.current.is_day ? 'day':'night') + '/' + api.forecast.current.condition.icon.substr(api.forecast.current.condition.icon.length- 7, 3) + '.svg')"
              alt="">
        </div>
        <div class="current-temperature">
          {{ Math.round(api.forecast.current.temp_c) }}<sup>°{{ unit.temp }}</sup>
        </div>
        <div class="current-dt">
          {{ dayOfWeek[date.getDay()] }},
          <span> {{ date.getHours() + ":" + date.getMinutes() }}</span>
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
        <LocationWidget :location="api.forecast.location.name" :key="updateKey"/>
      </footer>
    </div>

    <div class="main">
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
      <section v-if="tab" class="week">
        <div v-for="(day, i) in api.forecast.forecast.forecastday" :key="i" class="container">
          <div class="weekday"> {{ weekDate(day.date) }}</div>
          <div class="wh-main">
            <img
                :src="require('@/assets/icons/line/all/day/' + day.day.condition.icon.substr(day.day.condition.icon.length - 7, 3) + '.svg')"
                alt="">
          </div>
          <footer class="center">
            <span class="high"> {{ Math.round(api.forecast.forecast.forecastday[i].day.maxtemp_c) + "° " }} </span>

            <span class="low"> {{ Math.round(api.forecast.forecast.forecastday[i].day.mintemp_c) + "°" }} </span>
          </footer>
        </div>
      </section>
      <h1>Today's highlights</h1>
      <div class="wh-container">
        <div class="weather-highlights">
          <div class="container center uv-index">
            <h1>UV Index</h1>
            <img
                :src="require('@/assets/icons/line/all/uv-index-'+ api.forecast.forecast.forecastday[selected.day].day.uv +'.svg')"
                alt="">
          </div>
          <div class="container">
            <h1>Wind Status</h1>
            <div class="wh-main">
              <div>
                <img alt="" src="./assets/icons/line/all/windsock.svg">
                <div class="value">
                  {{ api.forecast.current.wind_kph }} <sub>{{ unit.wind }} </sub>
                </div>
              </div>
            </div>
          </div>
          <div class="container ">
            <h1>Sunrise & Sunset</h1>
            <div class="wh-main flex">
              <div class="sunrise">
                <img alt="" src="./assets/icons/line/all/sunrise.svg">
                {{ api.forecast.forecast.forecastday[selected.day].astro.sunrise }}
              </div>
              <div class="sunset">
                <img alt="" src="./assets/icons/line/all/sunset.svg">
                {{ api.forecast.forecast.forecastday[selected.day].astro.sunset }}
              </div>
            </div>
          </div>
          <div class="container">
            <h1>Humidity</h1>
            <div class="wh-main">
              <div class="value">
                {{ api.forecast.forecast.forecastday[selected.day].day.avghumidity }}%
              </div>
              <div class="slider">
                <div :style="{height: api.forecast.forecast.forecastday[selected.day].day.avghumidity  + '%'}"
                     class="slider-value"></div>
              </div>
            </div>
            <footer>Normal emoji</footer>
          </div>
          <div class="container">
            <h1>Visibility</h1>
            <div class="wh-main">
              <div class="value">
                {{ api.forecast.forecast.forecastday[selected.day].day.avgvis_km }} <sub>km</sub>
              </div>
            </div>
          </div>
          <div v-if="selected.day===0" class="container">
            <h1>Air Quality</h1>
            <div class="wh-main">
              <div class="value"> {{ api.forecast.current.air_quality["us-epa-index"] }}</div>
              <div class="slider">
                <div :style="{height: api.forecast.current.air_quality['us-epa-index'] *10 + '%'}"
                     class="slider-value"></div>
              </div>
            </div>
            <footer>
              {{ aqi[api.forecast.current.air_quality["us-epa-index"] - 1] }}
            </footer>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import CurrentWeatherProperty from "./components/current-weather-property.vue";
import LocationWidget from "./components/location-widget.vue";
import Button from "./components/button"
import {aKey} from "./secrets.json"

export default {
  name: "App",
  components: {
    CurrentWeatherProperty,
    LocationWidget,
    'v-button': Button,
  },
  data() {
    return {
      tab: true,
      myLocation: {
        coords: "",
        'my-location-active': false,
        "my-location-error": false,
        loading: false,
      },
      unit: {
        b: false,
        temp: "C",
        precipitation: "mm",
        wind: "kph"
      },
      selected: {
        b: false,
        day: 0
      },
      searchText: "",
      date: new Date(),
      updateKey: 0,
      api: {
        forecast: {
          "location": {
            "name": "Stockholm",
            "region": "Stockholms Lan",
            "country": "Sweden",
            "lat": 59.33,
            "lon": 18.05,
            "tz_id": "Europe/Stockholm",
            "localtime_epoch": 1624537266,
            "localtime": "2021-06-24 14:21"
          },
          "current": {
            "last_updated_epoch": 1624532400,
            "last_updated": "2021-06-24 13:00",
            "temp_c": 22.0,
            "temp_f": 71.6,
            "is_day": 1,
            "condition": {
              "text": "Partly cloudy",
              "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
              "code": 1003
            },
            "wind_mph": 4.3,
            "wind_kph": 6.8,
            "wind_degree": 90,
            "wind_dir": "E",
            "pressure_mb": 1017.0,
            "pressure_in": 30.5,
            "precip_mm": 0.1,
            "precip_in": 0.0,
            "humidity": 61,
            "cloud": 50,
            "feelslike_c": 24.6,
            "feelslike_f": 76.2,
            "vis_km": 10.0,
            "vis_miles": 6.0,
            "uv": 5.0,
            "gust_mph": 5.6,
            "gust_kph": 9.0,
            "air_quality": {
              "co": 180.1999969482422,
              "no2": 7.599999904632568,
              "o3": 51.5,
              "so2": 7.599999904632568,
              "pm2_5": 1.7000000476837158,
              "pm10": 3.0999999046325684,
              "us-epa-index": 1,
              "gb-defra-index": 1
            }
          },
          "forecast": {
            "forecastday": [
              {
                "date": "2021-06-24",
                "date_epoch": 1624492800,
                "day": {
                  "maxtemp_c": 22.6,
                  "maxtemp_f": 72.7,
                  "mintemp_c": 15.0,
                  "mintemp_f": 59.0,
                  "avgtemp_c": 18.8,
                  "avgtemp_f": 65.9,
                  "maxwind_mph": 8.3,
                  "maxwind_kph": 13.3,
                  "totalprecip_mm": 0.2,
                  "totalprecip_in": 0.01,
                  "avgvis_km": 10.0,
                  "avgvis_miles": 6.0,
                  "avghumidity": 74.0,
                  "daily_will_it_rain": 1,
                  "daily_chance_of_rain": "90",
                  "daily_will_it_snow": 0,
                  "daily_chance_of_snow": "0",
                  "condition": {
                    "text": "Patchy rain possible",
                    "icon": "//cdn.weatherapi.com/weather/64x64/day/176.png",
                    "code": 1063
                  },
                  "uv": 5.0
                },
                "astro": {
                  "sunrise": "03:32 AM",
                  "sunset": "10:09 PM",
                  "moonrise": "10:32 PM",
                  "moonset": "02:44 AM",
                  "moon_phase": "Waning Gibbous",
                  "moon_illumination": "97"
                },
                "hour": [
                  {
                    "time_epoch": 1624485600,
                    "time": "2021-06-24 00:00",
                    "temp_c": 15.5,
                    "temp_f": 59.9,
                    "is_day": 0,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/night/116.png",
                      "code": 1003
                    },
                    "wind_mph": 4.0,
                    "wind_kph": 6.5,
                    "wind_degree": 136,
                    "wind_dir": "SE",
                    "pressure_mb": 1018.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 88,
                    "cloud": 52,
                    "feelslike_c": 15.5,
                    "feelslike_f": 59.9,
                    "windchill_c": 15.5,
                    "windchill_f": 59.9,
                    "heatindex_c": 15.5,
                    "heatindex_f": 59.9,
                    "dewpoint_c": 13.5,
                    "dewpoint_f": 56.3,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 6.0,
                    "gust_kph": 9.7,
                    "uv": 1.0
                  },
                  {
                    "time_epoch": 1624489200,
                    "time": "2021-06-24 01:00",
                    "temp_c": 15.0,
                    "temp_f": 59.0,
                    "is_day": 0,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/night/116.png",
                      "code": 1003
                    },
                    "wind_mph": 4.5,
                    "wind_kph": 7.2,
                    "wind_degree": 138,
                    "wind_dir": "SE",
                    "pressure_mb": 1018.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 89,
                    "cloud": 54,
                    "feelslike_c": 15.0,
                    "feelslike_f": 59.0,
                    "windchill_c": 15.0,
                    "windchill_f": 59.0,
                    "heatindex_c": 15.0,
                    "heatindex_f": 59.0,
                    "dewpoint_c": 13.2,
                    "dewpoint_f": 55.8,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 6.7,
                    "gust_kph": 10.8,
                    "uv": 1.0
                  },
                  {
                    "time_epoch": 1624492800,
                    "time": "2021-06-24 02:00",
                    "temp_c": 15.2,
                    "temp_f": 59.4,
                    "is_day": 0,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/night/116.png",
                      "code": 1003
                    },
                    "wind_mph": 3.8,
                    "wind_kph": 6.1,
                    "wind_degree": 153,
                    "wind_dir": "SSE",
                    "pressure_mb": 1018.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 88,
                    "cloud": 52,
                    "feelslike_c": 15.2,
                    "feelslike_f": 59.4,
                    "windchill_c": 15.2,
                    "windchill_f": 59.4,
                    "heatindex_c": 15.2,
                    "heatindex_f": 59.4,
                    "dewpoint_c": 13.3,
                    "dewpoint_f": 55.9,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 5.6,
                    "gust_kph": 9.0,
                    "uv": 1.0
                  },
                  {
                    "time_epoch": 1624496400,
                    "time": "2021-06-24 03:00",
                    "temp_c": 15.4,
                    "temp_f": 59.7,
                    "is_day": 0,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/night/116.png",
                      "code": 1003
                    },
                    "wind_mph": 2.9,
                    "wind_kph": 4.7,
                    "wind_degree": 168,
                    "wind_dir": "SSE",
                    "pressure_mb": 1018.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 88,
                    "cloud": 50,
                    "feelslike_c": 15.4,
                    "feelslike_f": 59.7,
                    "windchill_c": 15.4,
                    "windchill_f": 59.7,
                    "heatindex_c": 15.4,
                    "heatindex_f": 59.7,
                    "dewpoint_c": 13.4,
                    "dewpoint_f": 56.1,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 4.3,
                    "gust_kph": 6.8,
                    "uv": 1.0
                  },
                  {
                    "time_epoch": 1624500000,
                    "time": "2021-06-24 04:00",
                    "temp_c": 15.6,
                    "temp_f": 60.1,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 2.2,
                    "wind_kph": 3.6,
                    "wind_degree": 183,
                    "wind_dir": "S",
                    "pressure_mb": 1018.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 88,
                    "cloud": 48,
                    "feelslike_c": 15.6,
                    "feelslike_f": 60.1,
                    "windchill_c": 15.6,
                    "windchill_f": 60.1,
                    "heatindex_c": 15.6,
                    "heatindex_f": 60.1,
                    "dewpoint_c": 13.5,
                    "dewpoint_f": 56.3,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 3.1,
                    "gust_kph": 5.0,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624503600,
                    "time": "2021-06-24 05:00",
                    "temp_c": 16.2,
                    "temp_f": 61.2,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 2.2,
                    "wind_kph": 3.6,
                    "wind_degree": 170,
                    "wind_dir": "S",
                    "pressure_mb": 1018.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 86,
                    "cloud": 47,
                    "feelslike_c": 16.2,
                    "feelslike_f": 61.2,
                    "windchill_c": 16.2,
                    "windchill_f": 61.2,
                    "heatindex_c": 16.2,
                    "heatindex_f": 61.2,
                    "dewpoint_c": 13.8,
                    "dewpoint_f": 56.8,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 3.1,
                    "gust_kph": 5.0,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624507200,
                    "time": "2021-06-24 06:00",
                    "temp_c": 16.8,
                    "temp_f": 62.2,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 2.5,
                    "wind_kph": 4.0,
                    "wind_degree": 158,
                    "wind_dir": "SSE",
                    "pressure_mb": 1018.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 84,
                    "cloud": 46,
                    "feelslike_c": 16.8,
                    "feelslike_f": 62.2,
                    "windchill_c": 16.8,
                    "windchill_f": 62.2,
                    "heatindex_c": 16.8,
                    "heatindex_f": 62.2,
                    "dewpoint_c": 14.0,
                    "dewpoint_f": 57.2,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 3.1,
                    "gust_kph": 5.0,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624510800,
                    "time": "2021-06-24 07:00",
                    "temp_c": 17.4,
                    "temp_f": 63.3,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 2.5,
                    "wind_kph": 4.0,
                    "wind_degree": 146,
                    "wind_dir": "SSE",
                    "pressure_mb": 1018.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 82,
                    "cloud": 45,
                    "feelslike_c": 17.4,
                    "feelslike_f": 63.3,
                    "windchill_c": 17.4,
                    "windchill_f": 63.3,
                    "heatindex_c": 17.4,
                    "heatindex_f": 63.3,
                    "dewpoint_c": 14.3,
                    "dewpoint_f": 57.7,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 3.1,
                    "gust_kph": 5.0,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624514400,
                    "time": "2021-06-24 08:00",
                    "temp_c": 18.9,
                    "temp_f": 66.0,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 2.7,
                    "wind_kph": 4.3,
                    "wind_degree": 143,
                    "wind_dir": "SE",
                    "pressure_mb": 1018.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 76,
                    "cloud": 39,
                    "feelslike_c": 18.9,
                    "feelslike_f": 66.0,
                    "windchill_c": 18.9,
                    "windchill_f": 66.0,
                    "heatindex_c": 19.8,
                    "heatindex_f": 67.6,
                    "dewpoint_c": 14.4,
                    "dewpoint_f": 57.9,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 3.4,
                    "gust_kph": 5.4,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624518000,
                    "time": "2021-06-24 09:00",
                    "temp_c": 20.5,
                    "temp_f": 68.9,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 2.9,
                    "wind_kph": 4.7,
                    "wind_degree": 140,
                    "wind_dir": "SE",
                    "pressure_mb": 1018.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 69,
                    "cloud": 34,
                    "feelslike_c": 20.5,
                    "feelslike_f": 68.9,
                    "windchill_c": 20.5,
                    "windchill_f": 68.9,
                    "heatindex_c": 22.2,
                    "heatindex_f": 72.0,
                    "dewpoint_c": 14.5,
                    "dewpoint_f": 58.1,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 3.4,
                    "gust_kph": 5.4,
                    "uv": 6.0
                  },
                  {
                    "time_epoch": 1624521600,
                    "time": "2021-06-24 10:00",
                    "temp_c": 22.0,
                    "temp_f": 71.6,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 3.1,
                    "wind_kph": 5.0,
                    "wind_degree": 137,
                    "wind_dir": "SE",
                    "pressure_mb": 1018.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 63,
                    "cloud": 29,
                    "feelslike_c": 24.6,
                    "feelslike_f": 76.3,
                    "windchill_c": 22.0,
                    "windchill_f": 71.6,
                    "heatindex_c": 24.6,
                    "heatindex_f": 76.3,
                    "dewpoint_c": 14.6,
                    "dewpoint_f": 58.3,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 3.6,
                    "gust_kph": 5.8,
                    "uv": 6.0
                  },
                  {
                    "time_epoch": 1624525200,
                    "time": "2021-06-24 11:00",
                    "temp_c": 22.2,
                    "temp_f": 72.0,
                    "is_day": 1,
                    "condition": {
                      "text": "Light rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/353.png",
                      "code": 1240
                    },
                    "wind_mph": 4.9,
                    "wind_kph": 7.9,
                    "wind_degree": 144,
                    "wind_dir": "SE",
                    "pressure_mb": 1018.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.1,
                    "precip_in": 0.0,
                    "humidity": 61,
                    "cloud": 38,
                    "feelslike_c": 24.6,
                    "feelslike_f": 76.3,
                    "windchill_c": 22.2,
                    "windchill_f": 72.0,
                    "heatindex_c": 24.6,
                    "heatindex_f": 76.3,
                    "dewpoint_c": 14.3,
                    "dewpoint_f": 57.7,
                    "will_it_rain": 0,
                    "chance_of_rain": "30",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 5.6,
                    "gust_kph": 9.0,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624528800,
                    "time": "2021-06-24 12:00",
                    "temp_c": 22.4,
                    "temp_f": 72.3,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 6.5,
                    "wind_kph": 10.4,
                    "wind_degree": 150,
                    "wind_dir": "SSE",
                    "pressure_mb": 1017.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.1,
                    "precip_in": 0.0,
                    "humidity": 59,
                    "cloud": 47,
                    "feelslike_c": 24.7,
                    "feelslike_f": 76.5,
                    "windchill_c": 22.4,
                    "windchill_f": 72.3,
                    "heatindex_c": 24.7,
                    "heatindex_f": 76.5,
                    "dewpoint_c": 14.0,
                    "dewpoint_f": 57.2,
                    "will_it_rain": 0,
                    "chance_of_rain": "60",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 7.6,
                    "gust_kph": 12.2,
                    "uv": 6.0
                  },
                  {
                    "time_epoch": 1624532400,
                    "time": "2021-06-24 13:00",
                    "temp_c": 22.6,
                    "temp_f": 72.7,
                    "is_day": 1,
                    "condition": {
                      "text": "Light rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/353.png",
                      "code": 1240
                    },
                    "wind_mph": 8.3,
                    "wind_kph": 13.3,
                    "wind_degree": 156,
                    "wind_dir": "SSE",
                    "pressure_mb": 1017.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.2,
                    "precip_in": 0.01,
                    "humidity": 57,
                    "cloud": 56,
                    "feelslike_c": 24.7,
                    "feelslike_f": 76.5,
                    "windchill_c": 22.6,
                    "windchill_f": 72.7,
                    "heatindex_c": 24.7,
                    "heatindex_f": 76.5,
                    "dewpoint_c": 13.7,
                    "dewpoint_f": 56.7,
                    "will_it_rain": 1,
                    "chance_of_rain": "90",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 9.6,
                    "gust_kph": 15.5,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624536000,
                    "time": "2021-06-24 14:00",
                    "temp_c": 22.4,
                    "temp_f": 72.3,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 8.1,
                    "wind_kph": 13.0,
                    "wind_degree": 164,
                    "wind_dir": "SSE",
                    "pressure_mb": 1017.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.1,
                    "precip_in": 0.0,
                    "humidity": 58,
                    "cloud": 48,
                    "feelslike_c": 24.7,
                    "feelslike_f": 76.5,
                    "windchill_c": 22.4,
                    "windchill_f": 72.3,
                    "heatindex_c": 24.7,
                    "heatindex_f": 76.5,
                    "dewpoint_c": 13.7,
                    "dewpoint_f": 56.7,
                    "will_it_rain": 0,
                    "chance_of_rain": "60",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 9.4,
                    "gust_kph": 15.1,
                    "uv": 6.0
                  },
                  {
                    "time_epoch": 1624539600,
                    "time": "2021-06-24 15:00",
                    "temp_c": 22.1,
                    "temp_f": 71.8,
                    "is_day": 1,
                    "condition": {
                      "text": "Light rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/353.png",
                      "code": 1240
                    },
                    "wind_mph": 7.8,
                    "wind_kph": 12.6,
                    "wind_degree": 171,
                    "wind_dir": "S",
                    "pressure_mb": 1017.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.1,
                    "precip_in": 0.0,
                    "humidity": 59,
                    "cloud": 40,
                    "feelslike_c": 24.6,
                    "feelslike_f": 76.3,
                    "windchill_c": 22.1,
                    "windchill_f": 71.8,
                    "heatindex_c": 24.6,
                    "heatindex_f": 76.3,
                    "dewpoint_c": 13.8,
                    "dewpoint_f": 56.8,
                    "will_it_rain": 0,
                    "chance_of_rain": "30",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 9.4,
                    "gust_kph": 15.1,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624543200,
                    "time": "2021-06-24 16:00",
                    "temp_c": 21.9,
                    "temp_f": 71.4,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 7.6,
                    "wind_kph": 12.2,
                    "wind_degree": 179,
                    "wind_dir": "S",
                    "pressure_mb": 1016.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 60,
                    "cloud": 32,
                    "feelslike_c": 21.9,
                    "feelslike_f": 71.4,
                    "windchill_c": 21.9,
                    "windchill_f": 71.4,
                    "heatindex_c": 24.6,
                    "heatindex_f": 76.3,
                    "dewpoint_c": 13.8,
                    "dewpoint_f": 56.8,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 9.2,
                    "gust_kph": 14.8,
                    "uv": 6.0
                  },
                  {
                    "time_epoch": 1624546800,
                    "time": "2021-06-24 17:00",
                    "temp_c": 21.4,
                    "temp_f": 70.5,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 6.9,
                    "wind_kph": 11.2,
                    "wind_degree": 178,
                    "wind_dir": "S",
                    "pressure_mb": 1016.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 62,
                    "cloud": 27,
                    "feelslike_c": 21.4,
                    "feelslike_f": 70.5,
                    "windchill_c": 21.4,
                    "windchill_f": 70.5,
                    "heatindex_c": 23.2,
                    "heatindex_f": 73.8,
                    "dewpoint_c": 14.0,
                    "dewpoint_f": 57.2,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 8.5,
                    "gust_kph": 13.7,
                    "uv": 6.0
                  },
                  {
                    "time_epoch": 1624550400,
                    "time": "2021-06-24 18:00",
                    "temp_c": 21.0,
                    "temp_f": 69.8,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 6.3,
                    "wind_kph": 10.1,
                    "wind_degree": 178,
                    "wind_dir": "S",
                    "pressure_mb": 1016.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 65,
                    "cloud": 23,
                    "feelslike_c": 21.0,
                    "feelslike_f": 69.8,
                    "windchill_c": 21.0,
                    "windchill_f": 69.8,
                    "heatindex_c": 21.9,
                    "heatindex_f": 71.4,
                    "dewpoint_c": 14.1,
                    "dewpoint_f": 57.4,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 8.1,
                    "gust_kph": 13.0,
                    "uv": 6.0
                  },
                  {
                    "time_epoch": 1624554000,
                    "time": "2021-06-24 19:00",
                    "temp_c": 20.5,
                    "temp_f": 68.9,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 5.6,
                    "wind_kph": 9.0,
                    "wind_degree": 178,
                    "wind_dir": "S",
                    "pressure_mb": 1015.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 67,
                    "cloud": 18,
                    "feelslike_c": 20.5,
                    "feelslike_f": 68.9,
                    "windchill_c": 20.5,
                    "windchill_f": 68.9,
                    "heatindex_c": 20.5,
                    "heatindex_f": 68.9,
                    "dewpoint_c": 14.3,
                    "dewpoint_f": 57.7,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 7.4,
                    "gust_kph": 11.9,
                    "uv": 6.0
                  },
                  {
                    "time_epoch": 1624557600,
                    "time": "2021-06-24 20:00",
                    "temp_c": 18.9,
                    "temp_f": 66.0,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 4.9,
                    "wind_kph": 7.9,
                    "wind_degree": 175,
                    "wind_dir": "S",
                    "pressure_mb": 1015.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 75,
                    "cloud": 18,
                    "feelslike_c": 18.9,
                    "feelslike_f": 66.0,
                    "windchill_c": 18.9,
                    "windchill_f": 66.0,
                    "heatindex_c": 18.9,
                    "heatindex_f": 66.0,
                    "dewpoint_c": 14.2,
                    "dewpoint_f": 57.6,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 7.4,
                    "gust_kph": 11.9,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624561200,
                    "time": "2021-06-24 21:00",
                    "temp_c": 17.3,
                    "temp_f": 63.1,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 4.3,
                    "wind_kph": 6.8,
                    "wind_degree": 172,
                    "wind_dir": "S",
                    "pressure_mb": 1016.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 82,
                    "cloud": 18,
                    "feelslike_c": 17.3,
                    "feelslike_f": 63.1,
                    "windchill_c": 17.3,
                    "windchill_f": 63.1,
                    "heatindex_c": 17.3,
                    "heatindex_f": 63.1,
                    "dewpoint_c": 14.0,
                    "dewpoint_f": 57.2,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 7.4,
                    "gust_kph": 11.9,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624564800,
                    "time": "2021-06-24 22:00",
                    "temp_c": 15.7,
                    "temp_f": 60.3,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 3.6,
                    "wind_kph": 5.8,
                    "wind_degree": 170,
                    "wind_dir": "S",
                    "pressure_mb": 1016.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 89,
                    "cloud": 17,
                    "feelslike_c": 15.7,
                    "feelslike_f": 60.3,
                    "windchill_c": 15.7,
                    "windchill_f": 60.3,
                    "heatindex_c": 15.7,
                    "heatindex_f": 60.3,
                    "dewpoint_c": 13.9,
                    "dewpoint_f": 57.0,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 7.4,
                    "gust_kph": 11.9,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624568400,
                    "time": "2021-06-24 23:00",
                    "temp_c": 15.5,
                    "temp_f": 59.9,
                    "is_day": 0,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/night/116.png",
                      "code": 1003
                    },
                    "wind_mph": 3.4,
                    "wind_kph": 5.4,
                    "wind_degree": 170,
                    "wind_dir": "S",
                    "pressure_mb": 1016.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 90,
                    "cloud": 18,
                    "feelslike_c": 15.6,
                    "feelslike_f": 60.1,
                    "windchill_c": 15.6,
                    "windchill_f": 60.1,
                    "heatindex_c": 15.5,
                    "heatindex_f": 59.9,
                    "dewpoint_c": 13.8,
                    "dewpoint_f": 56.8,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 6.9,
                    "gust_kph": 11.2,
                    "uv": 1.0
                  }
                ]
              },
              {
                "date": "2021-06-25",
                "date_epoch": 1624579200,
                "day": {
                  "maxtemp_c": 24.6,
                  "maxtemp_f": 76.3,
                  "mintemp_c": 15.0,
                  "mintemp_f": 59.0,
                  "avgtemp_c": 18.9,
                  "avgtemp_f": 66.0,
                  "maxwind_mph": 4.3,
                  "maxwind_kph": 6.8,
                  "totalprecip_mm": 0.5,
                  "totalprecip_in": 0.02,
                  "avgvis_km": 10.0,
                  "avgvis_miles": 6.0,
                  "avghumidity": 75.0,
                  "daily_will_it_rain": 0,
                  "daily_chance_of_rain": "69",
                  "daily_will_it_snow": 0,
                  "daily_chance_of_snow": "0",
                  "condition": {
                    "text": "Patchy rain possible",
                    "icon": "//cdn.weatherapi.com/weather/64x64/day/176.png",
                    "code": 1063
                  },
                  "uv": 6.0
                },
                "astro": {
                  "sunrise": "03:32 AM",
                  "sunset": "10:09 PM",
                  "moonrise": "11:33 PM",
                  "moonset": "03:31 AM",
                  "moon_phase": "Waning Gibbous",
                  "moon_illumination": "90"
                },
                "hour": [
                  {
                    "time_epoch": 1624572000,
                    "time": "2021-06-25 00:00",
                    "temp_c": 15.2,
                    "temp_f": 59.4,
                    "is_day": 0,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/night/116.png",
                      "code": 1003
                    },
                    "wind_mph": 3.1,
                    "wind_kph": 5.0,
                    "wind_degree": 171,
                    "wind_dir": "S",
                    "pressure_mb": 1015.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 91,
                    "cloud": 18,
                    "feelslike_c": 15.6,
                    "feelslike_f": 60.1,
                    "windchill_c": 15.6,
                    "windchill_f": 60.1,
                    "heatindex_c": 15.2,
                    "heatindex_f": 59.4,
                    "dewpoint_c": 13.7,
                    "dewpoint_f": 56.7,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 6.3,
                    "gust_kph": 10.1,
                    "uv": 1.0
                  },
                  {
                    "time_epoch": 1624575600,
                    "time": "2021-06-25 01:00",
                    "temp_c": 15.0,
                    "temp_f": 59.0,
                    "is_day": 0,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/night/116.png",
                      "code": 1003
                    },
                    "wind_mph": 2.9,
                    "wind_kph": 4.7,
                    "wind_degree": 171,
                    "wind_dir": "S",
                    "pressure_mb": 1015.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 91,
                    "cloud": 18,
                    "feelslike_c": 15.5,
                    "feelslike_f": 59.9,
                    "windchill_c": 15.5,
                    "windchill_f": 59.9,
                    "heatindex_c": 15.0,
                    "heatindex_f": 59.0,
                    "dewpoint_c": 13.6,
                    "dewpoint_f": 56.5,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 5.8,
                    "gust_kph": 9.4,
                    "uv": 1.0
                  },
                  {
                    "time_epoch": 1624579200,
                    "time": "2021-06-25 02:00",
                    "temp_c": 15.2,
                    "temp_f": 59.4,
                    "is_day": 0,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/night/116.png",
                      "code": 1003
                    },
                    "wind_mph": 2.7,
                    "wind_kph": 4.3,
                    "wind_degree": 162,
                    "wind_dir": "SSE",
                    "pressure_mb": 1015.0,
                    "pressure_in": 30.5,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 91,
                    "cloud": 23,
                    "feelslike_c": 15.5,
                    "feelslike_f": 59.9,
                    "windchill_c": 15.5,
                    "windchill_f": 59.9,
                    "heatindex_c": 15.2,
                    "heatindex_f": 59.4,
                    "dewpoint_c": 13.6,
                    "dewpoint_f": 56.5,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 5.1,
                    "gust_kph": 8.3,
                    "uv": 1.0
                  },
                  {
                    "time_epoch": 1624582800,
                    "time": "2021-06-25 03:00",
                    "temp_c": 15.3,
                    "temp_f": 59.5,
                    "is_day": 0,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/night/116.png",
                      "code": 1003
                    },
                    "wind_mph": 2.5,
                    "wind_kph": 4.0,
                    "wind_degree": 153,
                    "wind_dir": "SSE",
                    "pressure_mb": 1015.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 90,
                    "cloud": 28,
                    "feelslike_c": 15.5,
                    "feelslike_f": 59.9,
                    "windchill_c": 15.5,
                    "windchill_f": 59.9,
                    "heatindex_c": 15.3,
                    "heatindex_f": 59.5,
                    "dewpoint_c": 13.7,
                    "dewpoint_f": 56.7,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 4.5,
                    "gust_kph": 7.2,
                    "uv": 1.0
                  },
                  {
                    "time_epoch": 1624586400,
                    "time": "2021-06-25 04:00",
                    "temp_c": 15.5,
                    "temp_f": 59.9,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 2.2,
                    "wind_kph": 3.6,
                    "wind_degree": 144,
                    "wind_dir": "SE",
                    "pressure_mb": 1015.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 89,
                    "cloud": 33,
                    "feelslike_c": 15.5,
                    "feelslike_f": 59.9,
                    "windchill_c": 15.5,
                    "windchill_f": 59.9,
                    "heatindex_c": 15.5,
                    "heatindex_f": 59.9,
                    "dewpoint_c": 13.7,
                    "dewpoint_f": 56.7,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 3.8,
                    "gust_kph": 6.1,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624590000,
                    "time": "2021-06-25 05:00",
                    "temp_c": 16.7,
                    "temp_f": 62.1,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 2.5,
                    "wind_kph": 4.0,
                    "wind_degree": 128,
                    "wind_dir": "SE",
                    "pressure_mb": 1015.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 83,
                    "cloud": 41,
                    "feelslike_c": 16.7,
                    "feelslike_f": 62.1,
                    "windchill_c": 16.7,
                    "windchill_f": 62.1,
                    "heatindex_c": 16.7,
                    "heatindex_f": 62.1,
                    "dewpoint_c": 13.7,
                    "dewpoint_f": 56.7,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 3.6,
                    "gust_kph": 5.8,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624593600,
                    "time": "2021-06-25 06:00",
                    "temp_c": 18.0,
                    "temp_f": 64.4,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 2.5,
                    "wind_kph": 4.0,
                    "wind_degree": 112,
                    "wind_dir": "ESE",
                    "pressure_mb": 1014.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 77,
                    "cloud": 48,
                    "feelslike_c": 18.0,
                    "feelslike_f": 64.4,
                    "windchill_c": 18.0,
                    "windchill_f": 64.4,
                    "heatindex_c": 18.0,
                    "heatindex_f": 64.4,
                    "dewpoint_c": 13.7,
                    "dewpoint_f": 56.7,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 3.4,
                    "gust_kph": 5.4,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624597200,
                    "time": "2021-06-25 07:00",
                    "temp_c": 19.2,
                    "temp_f": 66.6,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 2.7,
                    "wind_kph": 4.3,
                    "wind_degree": 95,
                    "wind_dir": "E",
                    "pressure_mb": 1014.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 70,
                    "cloud": 56,
                    "feelslike_c": 19.2,
                    "feelslike_f": 66.6,
                    "windchill_c": 19.2,
                    "windchill_f": 66.6,
                    "heatindex_c": 19.2,
                    "heatindex_f": 66.6,
                    "dewpoint_c": 13.7,
                    "dewpoint_f": 56.7,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 3.1,
                    "gust_kph": 5.0,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624600800,
                    "time": "2021-06-25 08:00",
                    "temp_c": 19.8,
                    "temp_f": 67.6,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 2.5,
                    "wind_kph": 4.0,
                    "wind_degree": 87,
                    "wind_dir": "E",
                    "pressure_mb": 1014.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 68,
                    "cloud": 57,
                    "feelslike_c": 19.8,
                    "feelslike_f": 67.6,
                    "windchill_c": 19.8,
                    "windchill_f": 67.6,
                    "heatindex_c": 19.8,
                    "heatindex_f": 67.6,
                    "dewpoint_c": 13.6,
                    "dewpoint_f": 56.5,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 2.9,
                    "gust_kph": 4.7,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624604400,
                    "time": "2021-06-25 09:00",
                    "temp_c": 20.3,
                    "temp_f": 68.5,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 2.0,
                    "wind_kph": 3.2,
                    "wind_degree": 78,
                    "wind_dir": "ENE",
                    "pressure_mb": 1014.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 65,
                    "cloud": 58,
                    "feelslike_c": 20.3,
                    "feelslike_f": 68.5,
                    "windchill_c": 20.3,
                    "windchill_f": 68.5,
                    "heatindex_c": 20.3,
                    "heatindex_f": 68.5,
                    "dewpoint_c": 13.5,
                    "dewpoint_f": 56.3,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 2.5,
                    "gust_kph": 4.0,
                    "uv": 6.0
                  },
                  {
                    "time_epoch": 1624608000,
                    "time": "2021-06-25 10:00",
                    "temp_c": 20.9,
                    "temp_f": 69.6,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 1.8,
                    "wind_kph": 2.9,
                    "wind_degree": 69,
                    "wind_dir": "ENE",
                    "pressure_mb": 1014.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 62,
                    "cloud": 60,
                    "feelslike_c": 20.9,
                    "feelslike_f": 69.6,
                    "windchill_c": 20.9,
                    "windchill_f": 69.6,
                    "heatindex_c": 20.9,
                    "heatindex_f": 69.6,
                    "dewpoint_c": 13.4,
                    "dewpoint_f": 56.1,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 2.2,
                    "gust_kph": 3.6,
                    "uv": 6.0
                  },
                  {
                    "time_epoch": 1624611600,
                    "time": "2021-06-25 11:00",
                    "temp_c": 22.1,
                    "temp_f": 71.8,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 2.7,
                    "wind_kph": 4.3,
                    "wind_degree": 74,
                    "wind_dir": "ENE",
                    "pressure_mb": 1014.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 57,
                    "cloud": 52,
                    "feelslike_c": 22.4,
                    "feelslike_f": 72.3,
                    "windchill_c": 22.1,
                    "windchill_f": 71.8,
                    "heatindex_c": 22.4,
                    "heatindex_f": 72.3,
                    "dewpoint_c": 13.0,
                    "dewpoint_f": 55.4,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 3.1,
                    "gust_kph": 5.0,
                    "uv": 6.0
                  },
                  {
                    "time_epoch": 1624615200,
                    "time": "2021-06-25 12:00",
                    "temp_c": 23.4,
                    "temp_f": 74.1,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 3.4,
                    "wind_kph": 5.4,
                    "wind_degree": 80,
                    "wind_dir": "E",
                    "pressure_mb": 1014.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 52,
                    "cloud": 45,
                    "feelslike_c": 23.9,
                    "feelslike_f": 75.0,
                    "windchill_c": 23.4,
                    "windchill_f": 74.1,
                    "heatindex_c": 23.9,
                    "heatindex_f": 75.0,
                    "dewpoint_c": 12.7,
                    "dewpoint_f": 54.9,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 4.0,
                    "gust_kph": 6.5,
                    "uv": 6.0
                  },
                  {
                    "time_epoch": 1624618800,
                    "time": "2021-06-25 13:00",
                    "temp_c": 24.6,
                    "temp_f": 76.3,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 4.3,
                    "wind_kph": 6.8,
                    "wind_degree": 85,
                    "wind_dir": "E",
                    "pressure_mb": 1013.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 46,
                    "cloud": 38,
                    "feelslike_c": 25.4,
                    "feelslike_f": 77.7,
                    "windchill_c": 24.6,
                    "windchill_f": 76.3,
                    "heatindex_c": 25.4,
                    "heatindex_f": 77.7,
                    "dewpoint_c": 12.3,
                    "dewpoint_f": 54.1,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 4.9,
                    "gust_kph": 7.9,
                    "uv": 6.0
                  },
                  {
                    "time_epoch": 1624622400,
                    "time": "2021-06-25 14:00",
                    "temp_c": 23.3,
                    "temp_f": 73.9,
                    "is_day": 1,
                    "condition": {
                      "text": "Cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/119.png",
                      "code": 1006
                    },
                    "wind_mph": 4.0,
                    "wind_kph": 6.5,
                    "wind_degree": 111,
                    "wind_dir": "ESE",
                    "pressure_mb": 1013.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 51,
                    "cloud": 48,
                    "feelslike_c": 23.9,
                    "feelslike_f": 75.0,
                    "windchill_c": 23.3,
                    "windchill_f": 73.9,
                    "heatindex_c": 23.9,
                    "heatindex_f": 75.0,
                    "dewpoint_c": 12.4,
                    "dewpoint_f": 54.3,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 4.7,
                    "gust_kph": 7.6,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624626000,
                    "time": "2021-06-25 15:00",
                    "temp_c": 22.1,
                    "temp_f": 71.8,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 3.6,
                    "wind_kph": 5.8,
                    "wind_degree": 136,
                    "wind_dir": "SE",
                    "pressure_mb": 1013.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 55,
                    "cloud": 57,
                    "feelslike_c": 22.3,
                    "feelslike_f": 72.1,
                    "windchill_c": 22.1,
                    "windchill_f": 71.8,
                    "heatindex_c": 22.3,
                    "heatindex_f": 72.1,
                    "dewpoint_c": 12.5,
                    "dewpoint_f": 54.5,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 4.5,
                    "gust_kph": 7.2,
                    "uv": 6.0
                  },
                  {
                    "time_epoch": 1624629600,
                    "time": "2021-06-25 16:00",
                    "temp_c": 20.8,
                    "temp_f": 69.4,
                    "is_day": 1,
                    "condition": {
                      "text": "Cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/119.png",
                      "code": 1006
                    },
                    "wind_mph": 3.4,
                    "wind_kph": 5.4,
                    "wind_degree": 162,
                    "wind_dir": "SSE",
                    "pressure_mb": 1013.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 60,
                    "cloud": 66,
                    "feelslike_c": 20.8,
                    "feelslike_f": 69.4,
                    "windchill_c": 20.8,
                    "windchill_f": 69.4,
                    "heatindex_c": 20.8,
                    "heatindex_f": 69.4,
                    "dewpoint_c": 12.6,
                    "dewpoint_f": 54.7,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 4.3,
                    "gust_kph": 6.8,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624633200,
                    "time": "2021-06-25 17:00",
                    "temp_c": 19.9,
                    "temp_f": 67.8,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 3.6,
                    "wind_kph": 5.8,
                    "wind_degree": 191,
                    "wind_dir": "SSW",
                    "pressure_mb": 1013.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 69,
                    "cloud": 59,
                    "feelslike_c": 19.9,
                    "feelslike_f": 67.8,
                    "windchill_c": 19.9,
                    "windchill_f": 67.8,
                    "heatindex_c": 19.9,
                    "heatindex_f": 67.8,
                    "dewpoint_c": 13.7,
                    "dewpoint_f": 56.7,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 4.5,
                    "gust_kph": 7.2,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624636800,
                    "time": "2021-06-25 18:00",
                    "temp_c": 19.0,
                    "temp_f": 66.2,
                    "is_day": 1,
                    "condition": {
                      "text": "Cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/119.png",
                      "code": 1006
                    },
                    "wind_mph": 3.6,
                    "wind_kph": 5.8,
                    "wind_degree": 220,
                    "wind_dir": "SW",
                    "pressure_mb": 1013.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 78,
                    "cloud": 52,
                    "feelslike_c": 19.0,
                    "feelslike_f": 66.2,
                    "windchill_c": 19.0,
                    "windchill_f": 66.2,
                    "heatindex_c": 19.0,
                    "heatindex_f": 66.2,
                    "dewpoint_c": 14.9,
                    "dewpoint_f": 58.8,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 4.9,
                    "gust_kph": 7.9,
                    "uv": 4.0
                  },
                  {
                    "time_epoch": 1624640400,
                    "time": "2021-06-25 19:00",
                    "temp_c": 18.1,
                    "temp_f": 64.6,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 3.8,
                    "wind_kph": 6.1,
                    "wind_degree": 250,
                    "wind_dir": "WSW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 88,
                    "cloud": 44,
                    "feelslike_c": 18.1,
                    "feelslike_f": 64.6,
                    "windchill_c": 18.1,
                    "windchill_f": 64.6,
                    "heatindex_c": 18.1,
                    "heatindex_f": 64.6,
                    "dewpoint_c": 16.0,
                    "dewpoint_f": 60.8,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 5.1,
                    "gust_kph": 8.3,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624644000,
                    "time": "2021-06-25 20:00",
                    "temp_c": 17.8,
                    "temp_f": 64.0,
                    "is_day": 1,
                    "condition": {
                      "text": "Light rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/353.png",
                      "code": 1240
                    },
                    "wind_mph": 3.8,
                    "wind_kph": 6.1,
                    "wind_degree": 274,
                    "wind_dir": "W",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.2,
                    "precip_in": 0.01,
                    "humidity": 90,
                    "cloud": 63,
                    "feelslike_c": 17.8,
                    "feelslike_f": 64.0,
                    "windchill_c": 17.8,
                    "windchill_f": 64.0,
                    "heatindex_c": 17.8,
                    "heatindex_f": 64.0,
                    "dewpoint_c": 16.1,
                    "dewpoint_f": 61.0,
                    "will_it_rain": 0,
                    "chance_of_rain": "23",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 5.6,
                    "gust_kph": 9.0,
                    "uv": 4.0
                  },
                  {
                    "time_epoch": 1624647600,
                    "time": "2021-06-25 21:00",
                    "temp_c": 17.4,
                    "temp_f": 63.3,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 3.8,
                    "wind_kph": 6.1,
                    "wind_degree": 298,
                    "wind_dir": "WNW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.3,
                    "precip_in": 0.01,
                    "humidity": 93,
                    "cloud": 81,
                    "feelslike_c": 17.4,
                    "feelslike_f": 63.3,
                    "windchill_c": 17.4,
                    "windchill_f": 63.3,
                    "heatindex_c": 17.4,
                    "heatindex_f": 63.3,
                    "dewpoint_c": 16.2,
                    "dewpoint_f": 61.2,
                    "will_it_rain": 0,
                    "chance_of_rain": "46",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 5.8,
                    "gust_kph": 9.4,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624651200,
                    "time": "2021-06-25 22:00",
                    "temp_c": 17.1,
                    "temp_f": 62.8,
                    "is_day": 1,
                    "condition": {
                      "text": "Light rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/353.png",
                      "code": 1240
                    },
                    "wind_mph": 3.8,
                    "wind_kph": 6.1,
                    "wind_degree": 321,
                    "wind_dir": "NW",
                    "pressure_mb": 1013.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.5,
                    "precip_in": 0.02,
                    "humidity": 95,
                    "cloud": 100,
                    "feelslike_c": 17.1,
                    "feelslike_f": 62.8,
                    "windchill_c": 17.1,
                    "windchill_f": 62.8,
                    "heatindex_c": 17.1,
                    "heatindex_f": 62.8,
                    "dewpoint_c": 16.3,
                    "dewpoint_f": 61.3,
                    "will_it_rain": 0,
                    "chance_of_rain": "69",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 6.3,
                    "gust_kph": 10.1,
                    "uv": 4.0
                  },
                  {
                    "time_epoch": 1624654800,
                    "time": "2021-06-25 23:00",
                    "temp_c": 16.7,
                    "temp_f": 62.1,
                    "is_day": 0,
                    "condition": {
                      "text": "Light rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/night/353.png",
                      "code": 1240
                    },
                    "wind_mph": 4.9,
                    "wind_kph": 7.9,
                    "wind_degree": 326,
                    "wind_dir": "NNW",
                    "pressure_mb": 1013.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.5,
                    "precip_in": 0.02,
                    "humidity": 92,
                    "cloud": 84,
                    "feelslike_c": 16.7,
                    "feelslike_f": 62.1,
                    "windchill_c": 16.7,
                    "windchill_f": 62.1,
                    "heatindex_c": 16.7,
                    "heatindex_f": 62.1,
                    "dewpoint_c": 15.4,
                    "dewpoint_f": 59.7,
                    "will_it_rain": 1,
                    "chance_of_rain": "70",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 8.5,
                    "gust_kph": 13.7,
                    "uv": 1.0
                  }
                ]
              },
              {
                "date": "2021-06-26",
                "date_epoch": 1624665600,
                "day": {
                  "maxtemp_c": 22.0,
                  "maxtemp_f": 71.6,
                  "mintemp_c": 13.8,
                  "mintemp_f": 56.8,
                  "avgtemp_c": 17.4,
                  "avgtemp_f": 63.3,
                  "maxwind_mph": 11.4,
                  "maxwind_kph": 18.4,
                  "totalprecip_mm": 18.1,
                  "totalprecip_in": 0.71,
                  "avgvis_km": 8.8,
                  "avgvis_miles": 5.0,
                  "avghumidity": 82.0,
                  "daily_will_it_rain": 1,
                  "daily_chance_of_rain": "81",
                  "daily_will_it_snow": 0,
                  "daily_chance_of_snow": "0",
                  "condition": {
                    "text": "Moderate rain",
                    "icon": "//cdn.weatherapi.com/weather/64x64/day/302.png",
                    "code": 1189
                  },
                  "uv": 2.0
                },
                "astro": {
                  "sunrise": "03:33 AM",
                  "sunset": "10:08 PM",
                  "moonrise": "No moonrise",
                  "moonset": "04:43 AM",
                  "moon_phase": "Waning Gibbous",
                  "moon_illumination": "83"
                },
                "hour": [
                  {
                    "time_epoch": 1624658400,
                    "time": "2021-06-26 00:00",
                    "temp_c": 16.2,
                    "temp_f": 61.2,
                    "is_day": 0,
                    "condition": {
                      "text": "Light rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/night/353.png",
                      "code": 1240
                    },
                    "wind_mph": 5.8,
                    "wind_kph": 9.4,
                    "wind_degree": 330,
                    "wind_dir": "NNW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.6,
                    "precip_in": 0.02,
                    "humidity": 90,
                    "cloud": 68,
                    "feelslike_c": 16.2,
                    "feelslike_f": 61.2,
                    "windchill_c": 16.2,
                    "windchill_f": 61.2,
                    "heatindex_c": 16.2,
                    "heatindex_f": 61.2,
                    "dewpoint_c": 14.5,
                    "dewpoint_f": 58.1,
                    "will_it_rain": 1,
                    "chance_of_rain": "72",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 10.5,
                    "gust_kph": 16.9,
                    "uv": 1.0
                  },
                  {
                    "time_epoch": 1624662000,
                    "time": "2021-06-26 01:00",
                    "temp_c": 15.8,
                    "temp_f": 60.4,
                    "is_day": 0,
                    "condition": {
                      "text": "Light rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/night/353.png",
                      "code": 1240
                    },
                    "wind_mph": 6.9,
                    "wind_kph": 11.2,
                    "wind_degree": 334,
                    "wind_dir": "NNW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.6,
                    "precip_in": 0.02,
                    "humidity": 87,
                    "cloud": 52,
                    "feelslike_c": 15.8,
                    "feelslike_f": 60.4,
                    "windchill_c": 15.8,
                    "windchill_f": 60.4,
                    "heatindex_c": 15.8,
                    "heatindex_f": 60.4,
                    "dewpoint_c": 13.6,
                    "dewpoint_f": 56.5,
                    "will_it_rain": 1,
                    "chance_of_rain": "73",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 12.8,
                    "gust_kph": 20.5,
                    "uv": 1.0
                  },
                  {
                    "time_epoch": 1624665600,
                    "time": "2021-06-26 02:00",
                    "temp_c": 15.8,
                    "temp_f": 60.4,
                    "is_day": 0,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/night/116.png",
                      "code": 1003
                    },
                    "wind_mph": 7.4,
                    "wind_kph": 11.9,
                    "wind_degree": 336,
                    "wind_dir": "NNW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.4,
                    "precip_in": 0.02,
                    "humidity": 87,
                    "cloud": 54,
                    "feelslike_c": 15.8,
                    "feelslike_f": 60.4,
                    "windchill_c": 15.8,
                    "windchill_f": 60.4,
                    "heatindex_c": 15.8,
                    "heatindex_f": 60.4,
                    "dewpoint_c": 13.7,
                    "dewpoint_f": 56.7,
                    "will_it_rain": 0,
                    "chance_of_rain": "49",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 12.8,
                    "gust_kph": 20.5,
                    "uv": 1.0
                  },
                  {
                    "time_epoch": 1624669200,
                    "time": "2021-06-26 03:00",
                    "temp_c": 15.8,
                    "temp_f": 60.4,
                    "is_day": 0,
                    "condition": {
                      "text": "Light rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/night/353.png",
                      "code": 1240
                    },
                    "wind_mph": 7.6,
                    "wind_kph": 12.2,
                    "wind_degree": 338,
                    "wind_dir": "NNW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.2,
                    "precip_in": 0.01,
                    "humidity": 88,
                    "cloud": 56,
                    "feelslike_c": 15.8,
                    "feelslike_f": 60.4,
                    "windchill_c": 15.8,
                    "windchill_f": 60.4,
                    "heatindex_c": 15.8,
                    "heatindex_f": 60.4,
                    "dewpoint_c": 13.8,
                    "dewpoint_f": 56.8,
                    "will_it_rain": 0,
                    "chance_of_rain": "24",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 13.0,
                    "gust_kph": 20.9,
                    "uv": 1.0
                  },
                  {
                    "time_epoch": 1624672800,
                    "time": "2021-06-26 04:00",
                    "temp_c": 15.8,
                    "temp_f": 60.4,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 8.1,
                    "wind_kph": 13.0,
                    "wind_degree": 340,
                    "wind_dir": "NNW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.3,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 88,
                    "cloud": 58,
                    "feelslike_c": 15.8,
                    "feelslike_f": 60.4,
                    "windchill_c": 15.8,
                    "windchill_f": 60.4,
                    "heatindex_c": 15.8,
                    "heatindex_f": 60.4,
                    "dewpoint_c": 13.9,
                    "dewpoint_f": 57.0,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 13.0,
                    "gust_kph": 20.9,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624676400,
                    "time": "2021-06-26 05:00",
                    "temp_c": 16.4,
                    "temp_f": 61.5,
                    "is_day": 1,
                    "condition": {
                      "text": "Patchy rain possible",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/176.png",
                      "code": 1063
                    },
                    "wind_mph": 8.3,
                    "wind_kph": 13.3,
                    "wind_degree": 340,
                    "wind_dir": "NNW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.3,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 85,
                    "cloud": 66,
                    "feelslike_c": 16.4,
                    "feelslike_f": 61.5,
                    "windchill_c": 16.4,
                    "windchill_f": 61.5,
                    "heatindex_c": 16.4,
                    "heatindex_f": 61.5,
                    "dewpoint_c": 13.9,
                    "dewpoint_f": 57.0,
                    "will_it_rain": 0,
                    "chance_of_rain": "27",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 12.8,
                    "gust_kph": 20.5,
                    "uv": 4.0
                  },
                  {
                    "time_epoch": 1624680000,
                    "time": "2021-06-26 06:00",
                    "temp_c": 16.9,
                    "temp_f": 62.4,
                    "is_day": 1,
                    "condition": {
                      "text": "Partly cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/116.png",
                      "code": 1003
                    },
                    "wind_mph": 8.7,
                    "wind_kph": 14.0,
                    "wind_degree": 341,
                    "wind_dir": "NNW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.3,
                    "precip_mm": 0.1,
                    "precip_in": 0.0,
                    "humidity": 82,
                    "cloud": 73,
                    "feelslike_c": 16.9,
                    "feelslike_f": 62.4,
                    "windchill_c": 16.9,
                    "windchill_f": 62.4,
                    "heatindex_c": 16.9,
                    "heatindex_f": 62.4,
                    "dewpoint_c": 13.8,
                    "dewpoint_f": 56.8,
                    "will_it_rain": 0,
                    "chance_of_rain": "54",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 12.3,
                    "gust_kph": 19.8,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624683600,
                    "time": "2021-06-26 07:00",
                    "temp_c": 17.5,
                    "temp_f": 63.5,
                    "is_day": 1,
                    "condition": {
                      "text": "Patchy rain possible",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/176.png",
                      "code": 1063
                    },
                    "wind_mph": 8.9,
                    "wind_kph": 14.4,
                    "wind_degree": 342,
                    "wind_dir": "NNW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.3,
                    "precip_mm": 0.1,
                    "precip_in": 0.0,
                    "humidity": 79,
                    "cloud": 81,
                    "feelslike_c": 17.5,
                    "feelslike_f": 63.5,
                    "windchill_c": 17.5,
                    "windchill_f": 63.5,
                    "heatindex_c": 17.5,
                    "heatindex_f": 63.5,
                    "dewpoint_c": 13.8,
                    "dewpoint_f": 56.8,
                    "will_it_rain": 1,
                    "chance_of_rain": "81",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 12.1,
                    "gust_kph": 19.4,
                    "uv": 4.0
                  },
                  {
                    "time_epoch": 1624687200,
                    "time": "2021-06-26 08:00",
                    "temp_c": 19.0,
                    "temp_f": 66.2,
                    "is_day": 1,
                    "condition": {
                      "text": "Cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/119.png",
                      "code": 1006
                    },
                    "wind_mph": 9.8,
                    "wind_kph": 15.8,
                    "wind_degree": 344,
                    "wind_dir": "NNW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.3,
                    "precip_mm": 0.1,
                    "precip_in": 0.0,
                    "humidity": 71,
                    "cloud": 77,
                    "feelslike_c": 19.0,
                    "feelslike_f": 66.2,
                    "windchill_c": 19.0,
                    "windchill_f": 66.2,
                    "heatindex_c": 19.8,
                    "heatindex_f": 67.6,
                    "dewpoint_c": 13.4,
                    "dewpoint_f": 56.1,
                    "will_it_rain": 0,
                    "chance_of_rain": "54",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 12.5,
                    "gust_kph": 20.2,
                    "uv": 4.0
                  },
                  {
                    "time_epoch": 1624690800,
                    "time": "2021-06-26 09:00",
                    "temp_c": 20.5,
                    "temp_f": 68.9,
                    "is_day": 1,
                    "condition": {
                      "text": "Patchy rain possible",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/176.png",
                      "code": 1063
                    },
                    "wind_mph": 10.5,
                    "wind_kph": 16.9,
                    "wind_degree": 346,
                    "wind_dir": "NNW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.3,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 63,
                    "cloud": 74,
                    "feelslike_c": 20.5,
                    "feelslike_f": 68.9,
                    "windchill_c": 20.5,
                    "windchill_f": 68.9,
                    "heatindex_c": 22.2,
                    "heatindex_f": 72.0,
                    "dewpoint_c": 12.9,
                    "dewpoint_f": 55.2,
                    "will_it_rain": 0,
                    "chance_of_rain": "27",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 12.8,
                    "gust_kph": 20.5,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624694400,
                    "time": "2021-06-26 10:00",
                    "temp_c": 22.0,
                    "temp_f": 71.6,
                    "is_day": 1,
                    "condition": {
                      "text": "Cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/119.png",
                      "code": 1006
                    },
                    "wind_mph": 11.4,
                    "wind_kph": 18.4,
                    "wind_degree": 348,
                    "wind_dir": "NNW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.3,
                    "precip_mm": 0.0,
                    "precip_in": 0.0,
                    "humidity": 55,
                    "cloud": 70,
                    "feelslike_c": 24.5,
                    "feelslike_f": 76.1,
                    "windchill_c": 22.0,
                    "windchill_f": 71.6,
                    "heatindex_c": 24.5,
                    "heatindex_f": 76.1,
                    "dewpoint_c": 12.5,
                    "dewpoint_f": 54.5,
                    "will_it_rain": 0,
                    "chance_of_rain": "0",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 13.2,
                    "gust_kph": 21.2,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624698000,
                    "time": "2021-06-26 11:00",
                    "temp_c": 21.6,
                    "temp_f": 70.9,
                    "is_day": 1,
                    "condition": {
                      "text": "Light rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/353.png",
                      "code": 1240
                    },
                    "wind_mph": 11.0,
                    "wind_kph": 17.6,
                    "wind_degree": 233,
                    "wind_dir": "SW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.3,
                    "precip_mm": 0.7,
                    "precip_in": 0.03,
                    "humidity": 60,
                    "cloud": 73,
                    "feelslike_c": 21.6,
                    "feelslike_f": 70.9,
                    "windchill_c": 21.6,
                    "windchill_f": 70.9,
                    "heatindex_c": 23.3,
                    "heatindex_f": 73.9,
                    "dewpoint_c": 13.5,
                    "dewpoint_f": 56.3,
                    "will_it_rain": 0,
                    "chance_of_rain": "27",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 13.6,
                    "gust_kph": 22.0,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624701600,
                    "time": "2021-06-26 12:00",
                    "temp_c": 21.2,
                    "temp_f": 70.2,
                    "is_day": 1,
                    "condition": {
                      "text": "Cloudy",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/119.png",
                      "code": 1006
                    },
                    "wind_mph": 10.3,
                    "wind_kph": 16.6,
                    "wind_degree": 117,
                    "wind_dir": "ESE",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.3,
                    "precip_mm": 1.4,
                    "precip_in": 0.06,
                    "humidity": 66,
                    "cloud": 76,
                    "feelslike_c": 21.2,
                    "feelslike_f": 70.2,
                    "windchill_c": 21.2,
                    "windchill_f": 70.2,
                    "heatindex_c": 22.0,
                    "heatindex_f": 71.6,
                    "dewpoint_c": 14.6,
                    "dewpoint_f": 58.3,
                    "will_it_rain": 0,
                    "chance_of_rain": "53",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 14.3,
                    "gust_kph": 23.0,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624705200,
                    "time": "2021-06-26 13:00",
                    "temp_c": 20.8,
                    "temp_f": 69.4,
                    "is_day": 1,
                    "condition": {
                      "text": "Light rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/353.png",
                      "code": 1240
                    },
                    "wind_mph": 9.8,
                    "wind_kph": 15.8,
                    "wind_degree": 2,
                    "wind_dir": "N",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.3,
                    "precip_mm": 2.1,
                    "precip_in": 0.08,
                    "humidity": 72,
                    "cloud": 79,
                    "feelslike_c": 20.8,
                    "feelslike_f": 69.4,
                    "windchill_c": 20.8,
                    "windchill_f": 69.4,
                    "heatindex_c": 20.8,
                    "heatindex_f": 69.4,
                    "dewpoint_c": 15.6,
                    "dewpoint_f": 60.1,
                    "will_it_rain": 1,
                    "chance_of_rain": "80",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 14.8,
                    "gust_kph": 23.8,
                    "uv": 5.0
                  },
                  {
                    "time_epoch": 1624708800,
                    "time": "2021-06-26 14:00",
                    "temp_c": 19.8,
                    "temp_f": 67.6,
                    "is_day": 1,
                    "condition": {
                      "text": "Moderate or heavy rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/356.png",
                      "code": 1243
                    },
                    "wind_mph": 9.8,
                    "wind_kph": 15.8,
                    "wind_degree": 115,
                    "wind_dir": "ESE",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.4,
                    "precip_mm": 3.1,
                    "precip_in": 0.12,
                    "humidity": 77,
                    "cloud": 86,
                    "feelslike_c": 19.8,
                    "feelslike_f": 67.6,
                    "windchill_c": 19.8,
                    "windchill_f": 67.6,
                    "heatindex_c": 19.8,
                    "heatindex_f": 67.6,
                    "dewpoint_c": 15.6,
                    "dewpoint_f": 60.1,
                    "will_it_rain": 1,
                    "chance_of_rain": "79",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 9.0,
                    "vis_miles": 5.0,
                    "gust_mph": 15.9,
                    "gust_kph": 25.6,
                    "uv": 4.0
                  },
                  {
                    "time_epoch": 1624712400,
                    "time": "2021-06-26 15:00",
                    "temp_c": 18.9,
                    "temp_f": 66.0,
                    "is_day": 1,
                    "condition": {
                      "text": "Light rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/353.png",
                      "code": 1240
                    },
                    "wind_mph": 9.8,
                    "wind_kph": 15.8,
                    "wind_degree": 227,
                    "wind_dir": "SW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.4,
                    "precip_mm": 4.0,
                    "precip_in": 0.16,
                    "humidity": 82,
                    "cloud": 93,
                    "feelslike_c": 18.9,
                    "feelslike_f": 66.0,
                    "windchill_c": 18.9,
                    "windchill_f": 66.0,
                    "heatindex_c": 18.9,
                    "heatindex_f": 66.0,
                    "dewpoint_c": 15.7,
                    "dewpoint_f": 60.3,
                    "will_it_rain": 1,
                    "chance_of_rain": "79",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 8.0,
                    "vis_miles": 4.0,
                    "gust_mph": 17.2,
                    "gust_kph": 27.7,
                    "uv": 4.0
                  },
                  {
                    "time_epoch": 1624716000,
                    "time": "2021-06-26 16:00",
                    "temp_c": 17.9,
                    "temp_f": 64.2,
                    "is_day": 1,
                    "condition": {
                      "text": "Moderate or heavy rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/356.png",
                      "code": 1243
                    },
                    "wind_mph": 9.8,
                    "wind_kph": 15.8,
                    "wind_degree": 340,
                    "wind_dir": "NNW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.4,
                    "precip_mm": 5.0,
                    "precip_in": 0.2,
                    "humidity": 87,
                    "cloud": 100,
                    "feelslike_c": 17.9,
                    "feelslike_f": 64.2,
                    "windchill_c": 17.9,
                    "windchill_f": 64.2,
                    "heatindex_c": 17.9,
                    "heatindex_f": 64.2,
                    "dewpoint_c": 15.7,
                    "dewpoint_f": 60.3,
                    "will_it_rain": 1,
                    "chance_of_rain": "78",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 7.0,
                    "vis_miles": 4.0,
                    "gust_mph": 18.3,
                    "gust_kph": 29.5,
                    "uv": 4.0
                  },
                  {
                    "time_epoch": 1624719600,
                    "time": "2021-06-26 17:00",
                    "temp_c": 17.2,
                    "temp_f": 63.0,
                    "is_day": 1,
                    "condition": {
                      "text": "Torrential rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/359.png",
                      "code": 1246
                    },
                    "wind_mph": 9.2,
                    "wind_kph": 14.8,
                    "wind_degree": 336,
                    "wind_dir": "NNW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.4,
                    "precip_mm": 6.5,
                    "precip_in": 0.26,
                    "humidity": 89,
                    "cloud": 87,
                    "feelslike_c": 17.2,
                    "feelslike_f": 63.0,
                    "windchill_c": 17.2,
                    "windchill_f": 63.0,
                    "heatindex_c": 17.2,
                    "heatindex_f": 63.0,
                    "dewpoint_c": 15.4,
                    "dewpoint_f": 59.7,
                    "will_it_rain": 1,
                    "chance_of_rain": "72",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 5.7,
                    "vis_miles": 3.0,
                    "gust_mph": 17.9,
                    "gust_kph": 28.8,
                    "uv": 4.0
                  },
                  {
                    "time_epoch": 1624723200,
                    "time": "2021-06-26 18:00",
                    "temp_c": 16.4,
                    "temp_f": 61.5,
                    "is_day": 1,
                    "condition": {
                      "text": "Moderate or heavy rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/356.png",
                      "code": 1243
                    },
                    "wind_mph": 8.7,
                    "wind_kph": 14.0,
                    "wind_degree": 332,
                    "wind_dir": "NNW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.4,
                    "precip_mm": 8.1,
                    "precip_in": 0.32,
                    "humidity": 91,
                    "cloud": 74,
                    "feelslike_c": 16.4,
                    "feelslike_f": 61.5,
                    "windchill_c": 16.4,
                    "windchill_f": 61.5,
                    "heatindex_c": 16.4,
                    "heatindex_f": 61.5,
                    "dewpoint_c": 15.0,
                    "dewpoint_f": 59.0,
                    "will_it_rain": 0,
                    "chance_of_rain": "66",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 4.3,
                    "vis_miles": 2.0,
                    "gust_mph": 17.4,
                    "gust_kph": 28.1,
                    "uv": 4.0
                  },
                  {
                    "time_epoch": 1624726800,
                    "time": "2021-06-26 19:00",
                    "temp_c": 15.7,
                    "temp_f": 60.3,
                    "is_day": 1,
                    "condition": {
                      "text": "Torrential rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/359.png",
                      "code": 1246
                    },
                    "wind_mph": 8.1,
                    "wind_kph": 13.0,
                    "wind_degree": 327,
                    "wind_dir": "NNW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.4,
                    "precip_mm": 9.6,
                    "precip_in": 0.38,
                    "humidity": 94,
                    "cloud": 60,
                    "feelslike_c": 15.7,
                    "feelslike_f": 60.3,
                    "windchill_c": 15.7,
                    "windchill_f": 60.3,
                    "heatindex_c": 15.7,
                    "heatindex_f": 60.3,
                    "dewpoint_c": 14.7,
                    "dewpoint_f": 58.5,
                    "will_it_rain": 0,
                    "chance_of_rain": "60",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 3.0,
                    "vis_miles": 1.0,
                    "gust_mph": 17.0,
                    "gust_kph": 27.4,
                    "uv": 4.0
                  },
                  {
                    "time_epoch": 1624730400,
                    "time": "2021-06-26 20:00",
                    "temp_c": 15.1,
                    "temp_f": 59.2,
                    "is_day": 1,
                    "condition": {
                      "text": "Light rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/353.png",
                      "code": 1240
                    },
                    "wind_mph": 8.5,
                    "wind_kph": 13.7,
                    "wind_degree": 320,
                    "wind_dir": "NW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.4,
                    "precip_mm": 6.6,
                    "precip_in": 0.26,
                    "humidity": 94,
                    "cloud": 58,
                    "feelslike_c": 14.7,
                    "feelslike_f": 58.5,
                    "windchill_c": 14.7,
                    "windchill_f": 58.5,
                    "heatindex_c": 15.1,
                    "heatindex_f": 59.2,
                    "dewpoint_c": 14.1,
                    "dewpoint_f": 57.4,
                    "will_it_rain": 0,
                    "chance_of_rain": "63",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 5.3,
                    "vis_miles": 3.0,
                    "gust_mph": 16.6,
                    "gust_kph": 26.6,
                    "uv": 4.0
                  },
                  {
                    "time_epoch": 1624734000,
                    "time": "2021-06-26 21:00",
                    "temp_c": 14.4,
                    "temp_f": 57.9,
                    "is_day": 1,
                    "condition": {
                      "text": "Torrential rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/359.png",
                      "code": 1246
                    },
                    "wind_mph": 8.9,
                    "wind_kph": 14.4,
                    "wind_degree": 312,
                    "wind_dir": "NW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.4,
                    "precip_mm": 3.7,
                    "precip_in": 0.15,
                    "humidity": 94,
                    "cloud": 55,
                    "feelslike_c": 13.6,
                    "feelslike_f": 56.5,
                    "windchill_c": 13.6,
                    "windchill_f": 56.5,
                    "heatindex_c": 14.4,
                    "heatindex_f": 57.9,
                    "dewpoint_c": 13.6,
                    "dewpoint_f": 56.5,
                    "will_it_rain": 0,
                    "chance_of_rain": "67",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 7.7,
                    "vis_miles": 4.0,
                    "gust_mph": 16.1,
                    "gust_kph": 25.9,
                    "uv": 3.0
                  },
                  {
                    "time_epoch": 1624737600,
                    "time": "2021-06-26 22:00",
                    "temp_c": 13.8,
                    "temp_f": 56.8,
                    "is_day": 1,
                    "condition": {
                      "text": "Light rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/day/353.png",
                      "code": 1240
                    },
                    "wind_mph": 9.4,
                    "wind_kph": 15.1,
                    "wind_degree": 305,
                    "wind_dir": "NW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.7,
                    "precip_in": 0.03,
                    "humidity": 95,
                    "cloud": 53,
                    "feelslike_c": 12.6,
                    "feelslike_f": 54.7,
                    "windchill_c": 12.6,
                    "windchill_f": 54.7,
                    "heatindex_c": 13.8,
                    "heatindex_f": 56.8,
                    "dewpoint_c": 13.0,
                    "dewpoint_f": 55.4,
                    "will_it_rain": 0,
                    "chance_of_rain": "70",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 15.7,
                    "gust_kph": 25.2,
                    "uv": 3.0
                  },
                  {
                    "time_epoch": 1624741200,
                    "time": "2021-06-26 23:00",
                    "temp_c": 13.6,
                    "temp_f": 56.5,
                    "is_day": 0,
                    "condition": {
                      "text": "Light rain shower",
                      "icon": "//cdn.weatherapi.com/weather/64x64/night/353.png",
                      "code": 1240
                    },
                    "wind_mph": 9.6,
                    "wind_kph": 15.5,
                    "wind_degree": 307,
                    "wind_dir": "NW",
                    "pressure_mb": 1012.0,
                    "pressure_in": 30.4,
                    "precip_mm": 0.9,
                    "precip_in": 0.04,
                    "humidity": 94,
                    "cloud": 57,
                    "feelslike_c": 12.3,
                    "feelslike_f": 54.1,
                    "windchill_c": 12.3,
                    "windchill_f": 54.1,
                    "heatindex_c": 13.6,
                    "heatindex_f": 56.5,
                    "dewpoint_c": 12.7,
                    "dewpoint_f": 54.9,
                    "will_it_rain": 1,
                    "chance_of_rain": "76",
                    "will_it_snow": 0,
                    "chance_of_snow": "0",
                    "vis_km": 10.0,
                    "vis_miles": 6.0,
                    "gust_mph": 16.1,
                    "gust_kph": 25.9,
                    "uv": 1.0
                  }
                ]
              }
            ]
          },
          "alerts": {
            "alert": []
          }
        }
      }
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
    const aqi = [
      "Good 🌳",
      "Moderate 🤙",
      "Unhealthy for sensitive group 🤔",
      "Unhealthy 👎",
      "Very Unhealthy 🏥",
      "Hazardous ⚠"
    ];
    return {dayOfWeek, aqi}
  },
  created() {
    this.date = new Date(this.api.forecast.location.localtime);
  },
  mounted() {
    navigator.permissions.query({
      name: 'geolocation'
    }).then((result) => {
      if (result.state === 'denied') {
        this.myLocation["my-location-error"] = true;
      }
    })
  },
  methods: {
    currentLocation() {
      this.myLocation.loading = true;

      navigator.permissions.query({
        name: 'geolocation',
      }).then((result) => {
        if (result.state === 'granted') {
          report(result.state)
          this.myLocation["my-location-active"] = true;
          navigator.geolocation.getCurrentPosition(this.forecast)
        } else if (result.state === 'prompt') {
          console.log(this.myLocation.loading)
          report(result.state);
          navigator.geolocation.getCurrentPosition(this.forecast)
        } else if (result.state === 'denied') {
          report(result.state);
          this.myLocation["my-location-error"] = true
          let element = document.getElementById("geolocation")
          element.classList.remove('shake'); // reset animation
          void element.offsetWidth; // trigger reflow
          element.classList.add('shake'); // start animation

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

      function report(state) {
        console.log("Permission " + state)
      }
    },
    forecast(pos, event) {
      let q;
      if (pos instanceof Object) {
        let coords = pos.coords;
        q = coords.latitude + "," + coords.longitude
      } else {
        if (/\d/.test(pos)) {
          throw Error("Search contains numbers.")
        }
        event.target.blur();
        this.myLocation["my-location-active"] = false;
        q = encodeURI(pos.toLowerCase().trim());
      }
      let URL = "http://api.weatherapi.com/v1/forecast.json" +
          "?key=" + aKey +
          "&q=" + q +
          "&days=9&aqi=yes&alerts=yes";
      if (!navigator.language.includes("en")) {
        URL = URL + "&lang=" + navigator.language
      }
      fetch(URL).then((res) => {
        if (res.status === 200) {
          res
              .json()
              .then((data) => {
                console.log(data);
                this.api.forecast = data;
                this.date = new Date(this.api.forecast.location.localtime);
                this.updateKey += 1;
              })
              .catch((error) => {
                console.log(error);
              })
        } else {
          res.json().then((err) => {
            console.error(err.error.message)
          })
        }
      })
    },
    weekDate(dayDate) {
      let d = new Date(dayDate);
      if (d.getDate() === this.date.getDate()) {
        return "Today"
      } else {
        return this.dayOfWeek[d.getDay()]
      }
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
      if (!this.api.forecast) return null
      let arr = [];
      if (this.api.forecast.current.condition.text.includes('snow')) {
        let snow = {
          icon: "snowflake.svg",
          value: "Snow — " + this.api.forecast.current.precip_mm,
          unit: "mm",
        };
        arr.push(snow);
      } else if (this.api.forecast.current.condition.text.includes('rain')) {
        let rain = {
          icon: "raindrops.svg",
          value: "Rain — " + this.api.forecast.current.precip_mm,
          unit: "mm",
        };
        arr.push(rain);
      }
      if (this.api.forecast.current.cloud > -1) {
        let clouds = {
          icon: "cloudy.svg",
          value: "Cloudiness — " + this.api.forecast.current.cloud,
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


.main {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-flow: column;
  box-sizing: border-box;
  max-height: 100vh;

  nav {
    flex: 0 0 auto;
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

  & > section {
    flex: 1 0 auto;
  }

  .week {
    flex: 0 0 auto;
    display: flex;
    flex-flow: row;
    flex-wrap: nowrap;

    .container {
      & > div {
        text-align: center;
        color: $primary-text-color;
      }

      &.weekday {

      }
    }

  }

  & > h1 {
    flex: 0 0 70px;
    display: flex;
    align-items: center;
    margin: 0;
  }

  .wh-container {
    display: flex;
    flex-flow: column;

    .weather-highlights {
      max-height: 57vh;
      display: flex;
      flex-wrap: wrap;
      flex-direction: row;
      align-content: center;
      justify-content: space-between;

      & > * {
        width: 30%;
        height: 50%;
      }

      .container {
        & > .wh-main {
          display: flex;
          flex-flow: row;
          justify-content: space-between;
          flex: 1 1 auto;

          .value {
            padding: 0.5em 0;
            font-size: xx-large;
          }

          &.flex {
            flex-flow: column;
            justify-content: space-around;

            div {
              height: 50%;

              img {
                max-height: 100%;
                max-width: 30%;
                height: auto;
                width: auto;
              }
            }
          }

          & > div {
            &.slider {
              border-radius: 25px;
              border: $accent-color solid 2px;
              width: 1.5em;
              align-items: flex-end;

              .slider-value {
                background-color: #2985C7;
                width: 100%;
                box-sizing: border-box;
                border-radius: 8px;
                margin: 4px;
                transition: height 1s;
              }
            }

            & > img {
              height: 4em;
            }

            display: flex;
            align-items: center;
          }
        }
      }
    }

    h1 {
      font-size: x-large;
    }


  }
}

.uv-index {
  align-self: center;

  img {
    max-height: 100%;
    max-width: 100%;
    width: auto;
    height: auto;
  }
}

.container {
  display: flex;
  flex-flow: column;
  padding: 1em;
  margin: 0.5em;
  overflow: hidden;
  box-sizing: border-box;
  background-color: $primary-color;
  border-radius: 15px;

  &.center {
    text-align: center;
  }

  h1 {
    text-align: left;
    color: $accent-text-color;
    margin: 0;
    font-size: 1em;
    font-weight: 400;
    flex: 0 1 30px;
  }

  footer {
    flex: 0 1 30px;
  }
}

.divider {
  margin: 1em auto;
  display: block;
  width: 100%;
  height: 1px;
  background-color: $accent-color;
}

.shake {
  animation: shake 0.82s cubic-bezier(.36, .07, .19, .97) both;
}


@keyframes shake {
  10%, 90% {
    transform: translate3d(-1px, 0, 0);
  }

  20%, 80% {
    transform: translate3d(2px, 0, 0);
  }

  30%, 50%, 70% {
    transform: translate3d(-4px, 0, 0);
  }

  40%, 60% {
    transform: translate3d(4px, 0, 0);
  }
}
</style>
