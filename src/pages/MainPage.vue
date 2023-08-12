<template>
  <main class="content container">
    <div class="content__top content__top--catalog">
      <h1 class="content__title">
        Каталог
      </h1>
      <span class="content__info">
        {{ countProducts }} товара
      </span>
    </div>

    <div class="content__catalog">

      <ProductFilter :price-from.sync="filterPriceFrom" :price-to.sync="filterPriceTo"
      :category-id.sync="filterCatId" :color-id.sync="filterColorId"/>

      <section class="catalog">

        <div style="display: flex; flex-direction: column; align-items: center; margin: 0 auto; " v-if="productsLoading">
          <div style="display: block; width: 200px; height: 200px;">
            <img style="width: 100%; height: 100%;" src="img/animation/loading-6.gif" alt="Animation">
          </div>
          <p>Загрузка товаров...</p>
        </div>
        <div v-if="productsLoadingFailed">Произошла ошибка при загрузка товаров
          <button @click.prevent="loadProducts">Повторить запрос</button>
        </div>

        <ProductList :products="products" />

        <BasePagination v-model="page" :count="countProducts" :per-page="productsPerPage" />

      </section>

    </div>
  </main>
</template>

<script>

import ProductList from '@/components/ProductList.vue';
import BasePagination from '@/components/BasePagination.vue';
import ProductFilter from '@/components/ProductFilter.vue';
import axios from 'axios';
import {API_BASE_URL} from '@/config';

export default {
  components: { ProductList, BasePagination, ProductFilter },
  data() {
    return {
      filterPriceFrom: 0,
      filterPriceTo: 0,
      filterCatId: 0,
      filterColorId: 0,
      page: 1,
      productsPerPage: 9,

      productsData: null,
      colorsData: null,

      productsLoading: false,
      productsLoadingFailed: false,
    };
  },
  computed: {
    products() {
      return this.productsData ? this.productsData.items.map(product => {
        return {
          ...product,
          image: product.image.file.url,
        }
      }) : [];
    },
    countProducts() {
      return this.productsData ? this.productsData.pagination.total : 0;
    },
  },
  methods: {

    loadProducts() {
      this.productsLoading = true,
      this.productsLoadingFailed = false,
      clearTimeout(this.loadProductsTimer);
      this.loadProductsTimer = setTimeout(() => {
        axios.get(API_BASE_URL + `/api/products`, {
        params: {
          page: this.page,
          limit: this.productsPerPage,
          categoryId: this.filterCatId,
          minPrice: this.filterPriceFrom,
          maxPrice: this.filterPriceTo,
          colorId: this.filterColorId,
        }
      })
          .then(response => this.productsData = response.data)
          .catch(() => this.productsLoadingFailed = true)
          .then(() => this.productsLoading = false);

      }, 2000);
    },
  },
  watch: {
    page() {
      this.loadProducts();
    },
    filterPriceFrom() {
      this.loadProducts();
    },
    filterPriceTo() {
      this.loadProducts();
    },
    filterCatId() {
      this.loadProducts();
    },
    filterColorId() {
      this.loadProducts();
    },

  },
  created() {
    this.loadProducts();
    },
};
</script>
