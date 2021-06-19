<template>
  <nav>
    <div class="logo">Weather App</div>
    <div class="search-bar">
      <input
        type="text"
        placeholder="search"
        v-model="searchText"
        @input="$emit('update:modelValue', $event.target.value)"
        autocomplete="off"
        maxlength="100"
      />
      <span class="material-icons-outlined"> search </span>
    </div>
    <div class="temp-unit">
      <span v-bind:class="{ active: !tempUnit }">C°</span>
      /
      <span v-bind:class="{ active: tempUnit }">F°</span>
    </div>
  </nav>
</template>

<script>
import { ref } from "@vue/reactivity";
export default {
  name: "Navbar",
  props: {
    unit: Boolean,
  },
  data() {
    return {
      tempUnit: false,
    };
  },
  setup() {
    const input = ref("");
    return { input };
  },
};
</script>

<style lang="scss" scoped>
@import "../styles/_globals.scss";
nav {
  display: block;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background-color: $primary-color;
  color: $primary-color-text;
  padding: 20px 2rem;
  .logo {
    font-weight: 800;
    font-size: 18px;
  }
  .search-bar {
    display: flex;
    align-items: center;
    width: 50%;
    input {
      font-family: inherit;
      font-size: 1em;
      padding: 7px 10px;
      outline: none;
      border: none;
      border-bottom: 2px solid $primary-color-text;
      background-color: $primary-color;
      color: $primary-color-text;
      width: 90%; // change this
      &::placeholder {
        color: $primary-color-text;
      }
      &:focus {
        &::placeholder {
          color: $primary-color-light;
        }
      }
    }
  }
  .temp-unit {
    cursor: default;
    color: $divider-color;
    span {
      padding: 0 0.2em;
      &.active {
        color: $primary-color-text;
      }
      &:hover {
        color: $primary-color-text;
      }
    }
    &:hover {
      .active {
        color: $divider-color;
        &:hover {
          color: $primary-color-text;
        }
      }
    }
  }
}
</style>