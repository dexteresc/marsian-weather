<template>
  <nav>
    <div class="logo">Weather App</div>
    <div class="searchbar">
      <input
        type="text"
        placeholder="search"
        v-model="searchText"
        @input="$emit('update:modelValue', $event.target.value)"
      />
      <span class="material-icons-outlined"> search </span>
    </div>
    <div class="sub-links">
      <span class="material-icons-outlined"> account_circle </span>
    </div>
  </nav>
</template>

<script>
import { ref } from "@vue/reactivity";
export default {
  name: "Navbar",
  data() {
    return {
    };
  },
  setup() {
    const input = ref("");
    return { input };
  },
};
</script>

<style lang="scss" scoped>
nav {
  display: block;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background-color: beige;
  padding: 20px 2rem;
}
</style>