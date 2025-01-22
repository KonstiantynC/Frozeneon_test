<template>
  <div class="product-list">
    <input
      v-model="searchQuery"
      class="search-input"
      @input="handleSearch"
    />

    <div class="list-container" @scroll="handleScroll">
      <div v-for="product in visibleProducts" :key="product.id" class="product-item">
        <img :src="product.thumbnail" :alt="product.title" class="product-image" />
        <div class="product-info">
          <h3>{{ product.title }}</h3>
          <p>{{ product.description }}</p>
          <p class="price">${{ product.price }}</p>
        </div>
      </div>

      <div v-if="visibleProducts.length === 0" class="no-results">
        No products found.
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { ref, computed, onMounted } from 'vue';

interface Product {
  id: number;
  title: string;
  description: string;
  price: number;
  thumbnail: string;
}

export default {
  name: 'ProductList',
  setup() {
    const products = ref<Product[]>([]);
    const searchQuery = ref('');
    const loadedCount = ref(20);
    const isSearching = ref(false);

    const filteredProducts = computed(() => {
      if (!searchQuery.value) {
        return products.value;
      }
      return products.value.filter((product) =>
        product.title.toLowerCase().includes(searchQuery.value.toLowerCase())
      );
    });

    const visibleProducts = computed(() => {
      return filteredProducts.value.slice(0, loadedCount.value);
    });

    const loadProducts = async () => {
      const response = await fetch('https://dummyjson.com/products?limit=100');
      const data = await response.json();
      products.value = data.products;
    };

    const handleScroll = (event: Event) => {
      if (isSearching.value) return;

      const target = event.target as HTMLElement;
      if (target.scrollTop + target.clientHeight >= target.scrollHeight - 100) {
        loadedCount.value += 20;
      }
    };

    const handleSearch = () => {
      isSearching.value = true;
      loadedCount.value = 20;
    };

    onMounted(() => {
      loadProducts();
    });

    return {
      searchQuery,
      visibleProducts,
      handleScroll,
      handleSearch,
    };
  },
};
</script>

<style scoped>


.product-list {
  max-width: 700px;
  margin: 0 auto;
  padding: 20px;
}

.search-input {
  width: 100%;
  padding: 10px;
  margin-bottom: 20px;
  font-size: 16px;
  border-radius: 6px;
}



.list-container {
  max-height: 600px;
  overflow-y: auto;
  border: 1px solid #ccc;
  border-radius: 8px;
}

.product-item {
  display: flex;
  padding: 10px;
  border-bottom: 1px solid #eee;
}

.product-image {
  width: 100px;
  height: 100px;
  object-fit: cover;
  margin-right: 20px;
  border-radius: 8px;
}

.product-info {
  flex: 1;
}

.price {
  color: green;
  font-weight: bold;
}
</style>
