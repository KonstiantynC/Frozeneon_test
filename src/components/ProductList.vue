<template>
  <div class="product-list">
    <input
      v-model="searchQuery"
      placeholder="Search products..."
      class="search-input"
      @input="handleSearch"
    />

    <div class="list-container" @scroll="handleScroll">
      <transition-group name="fade" tag="div">
        <div v-for="product in visibleProducts" :key="product.id" class="product-item">
          <img :src="product.thumbnail" :alt="product.title" class="product-image" />
          <div class="product-info">
            <h3 class="title">{{ product.title }}</h3>
            <p class="description">
              {{ truncateDescription(product) }}
              <span
                v-if="product.description.length > 60"
                class="read-more"
                @click="toggleDescription(product.id)"
              >
                ...
              </span>
            </p>
            <div class="brand_and_discount">
              <p class="discount" v-if="product.discountPercentage">Discount {{ product.discountPercentage }} %</p>
              <p class="brand">{{ product.brand }}</p>
            </div>
            <div class="price_rating_category">
              <p class="price">${{ product.price }} </p>
              <div class="rating_item">
                <div class="star"></div>
                <p class="rating"> {{ product.rating }}</p>
              </div>
              <p class="category"> {{ product.category }}</p>
            </div>
          </div>
        </div>

        <div v-if="visibleProducts.length === 0" key="no-results" class="no-results">
          No products found
        </div>
      </transition-group>
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
  discountPercentage: number;
  rating: number;
  brand: string;
  category: string;
  thumbnail: string;
}

export default {
  name: 'ProductList',
  setup() {
    const products = ref<Product[]>([]);
    const searchQuery = ref('');
    const loadedCount = ref(20);
    const isSearching = ref(false);
    const expandedDescriptions = ref<number | null>(null);

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
      console.log(products.value);
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

    const truncateDescription = (product: Product) => {
      if (expandedDescriptions.value === product.id) {
        return product.description;
      }
      return product.description.length > 60 ? product.description.slice(0, 60) : product.description;
    };

    const toggleDescription = (productId: number) => {
      expandedDescriptions.value = expandedDescriptions.value === productId ? null : productId;
    };

    onMounted(() => {
      loadProducts();
    });

    return {
      searchQuery,
      visibleProducts,
      handleScroll,
      handleSearch,
      truncateDescription,
      toggleDescription,
    };
  },
};
</script>

<style scoped>

.product-list {
  width: 700px;
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
  cursor: pointer;
  margin-bottom: 18px;
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

/* card style */
.title {
  font-size: 19px;
  font-weight: bold;
}

.brand_and_discount {
  display: flex;
  flex-direction: row;
  align-items: flex-end;
  margin-top: 6px;
}

.discount {
  font-weight: 500;
  color: red;
  margin-right: 30px;
  padding-bottom: 3px;
}

.brand {
  font-size: 19px;
  font-weight: bold;
  color: #309082;
}

.price_rating_category {
  display: flex;
  flex-direction: row;
  align-items: center;
  margin-top: 6px;
}

.price {
  color: #309082;
  font-weight: bold;
  margin-right: 30px;
  font-size: 18px;
}

.rating_item {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
}

.rating {
  color: #309082;
  margin-right: 30px;
  font-weight: bold;
  font-size: 18px;
}

.category {
  background-color:#0c2c17;
  color: #39d663;
  padding: 2px 12px;
  border-radius: 13px;
  font-weight: 500;
}



.no-results {
  display: flex;
  justify-content: center;
  width: 650px;
  font-size: 16px;
  font-weight: 500;
}
</style>
