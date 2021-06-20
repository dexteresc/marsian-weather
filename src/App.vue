<template>
  <Navbar v-model="searchText" v-on:tempUnit="changeUnit" :unit="unit" />
  {{searchText}}
  <Weather />
</template>

<script>
import { ref } from "vue";
import Navbar from "./components/Navbar.vue";
import Weather from "./components/Weather.vue";

export default {
  name: "App",
  components: {
    Navbar,
    Weather,
  },
  data() {
    return {
      unit: false,
    };
  },
  setup() {
    const searchText = ref("");
    return { searchText };
  },
  methods: {
    changeUnit(tempUnit) {
      console.log(this.unit, "this ran", tempUnit);
      this.unit = tempUnit;
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
