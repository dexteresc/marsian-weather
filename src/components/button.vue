<template>
  <button>
    <slot></slot>
  </button>
</template>
<script>
export default {
  name: "Button",
  props: {
    disabled: Boolean,
  },
  methods: {
    errorAnimation() {
      const element = document.getElementsByClassName('error')

      element.classList.remove('classname'); // reset animation
      void element.offsetWidth; // trigger reflow
      element.classList.add('classname'); // start animation
    }
  }
}
</script>
<style lang="scss" scoped>
@import "../styles/_globals.scss";

button {
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-basis: 40px;
  height: 40px;
  flex-grow: 0;
  flex-shrink: 0;
  padding: 1em;
  border: 0;
  border-radius: 25px;

  font-family: Avenir, Helvetica, Arial, sans-serif;
  color: $primary-text-color;
  background-color: $primary-color;

  &.primary {
    color: $primary-color-text;
    background-color: $primary-text-color;
  }

  &.accent {
    color: $primary-text-color;
    background-color: $accent-color;

  }

  &.rounded {
    border-radius: 50%;
    padding: 0;
  }

  &.active {
    color: blue;
  }
}




</style>