<template>
  <nav>
    <div class="logo">Weather App</div>
    <div class="search-bar">
      <input
        type="text"
        placeholder="search"
        @input="changeSearchText($event.target.value)"
        autocomplete="off"
        maxlength="100"
        @keypress.enter="$emit('searchClick')"
      />
      <span @click="$emit('searchClick')" class="material-icons-outlined">
        search
      </span>
    </div>
    <div class="temp-unit">
      <span :class="{ active: !unit }" @click="$emit('tempUnit', false)"
        >C°</span
      >
      /
      <span :class="{ active: unit }" @click="$emit('tempUnit', true)">F°</span>
    </div>
  </nav>
</template>

<script>
export default {
  name: "Navbar",
  emits: ["tempUnit", "update:modelValue", "searchClick"],
  props: {
    unit: Boolean,
    searchText: String,
  },
  methods: {
    changeSearchText(value) {
      this.$emit("update:modelValue", value);
    },
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
      cursor: pointer;
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