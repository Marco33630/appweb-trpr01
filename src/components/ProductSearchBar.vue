<script setup lang="ts">
import { ref, defineEmits } from 'vue';

const searchQuery = ref('');
const emit = defineEmits(['update-search']);

function onInput() {
  emit('update-search', searchQuery.value);
}
</script>

<template>
  <div class="searchBar">
    <input
      type="text"
      placeholder="Search products..."
      v-model="searchQuery"
      @input="onInput"
    />
  </div>
</template>

<style scoped>
  .searchBar {
    margin-bottom: 16px;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 5px;
    padding: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    border: 1px solid #ddd;
    background-color: #f9f9f9;
  }

  .searchBar input {
    width: 100%;
    padding: 8px;
    border: none;
    border-radius: 4px;
    outline: none;
    font-size: 16px;
  }

  .searchBar input::placeholder {
    color: #aaa;
  }
</style>