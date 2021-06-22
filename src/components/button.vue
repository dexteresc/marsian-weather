<template>
  <button>
    <slot></slot>
  </button>
</template>
<script>
export default {
  name: "Button",
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
  &.error {
    color: #f44336;
    &:active {
      animation: shake 0.82s cubic-bezier(.36, .07, .19, .97) both;
      backface-visibility: hidden;
    }
  }
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