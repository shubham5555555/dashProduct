<!-- src/App.vue -->
<template>
  <div id="app">
    <ProductList />
  </div>
</template>

<script>
import ProductList from './component/ProductList.vue';

export default {
  name: 'App',
  components: {
    ProductList
  }
};
</script>

<style>
body {
  background-color: #f8f9fa;
}
</style>
