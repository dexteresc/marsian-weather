<template>
  <div class="current-weather-property">
    <img :src="imgURL" alt=""/>
    <div class="value">{{ value }}{{ unit }}</div>
  </div>
</template>

<script>
export default {
  name: "CurrentWeatherProperty",
  props: {
    icon: String,
    value: String,
    unit: String,
  },
  computed: {
    imgURL() {
      if (this.icon) {
        let i
        try {
          i = require("@/assets/icons/line/all/" + this.icon);
        } catch (e) {
          console.error(e)
          i = require("@/assets/icons/line/all/not-available.svg")
        }
        return i
      } else return undefined;
    },
  },
};
</script>

<style lang="scss" scoped>
.current-weather-property {
  display: flex;
  align-items: center;
  font-size: 1em;
}

img {
  width: 2em;
}
</style>