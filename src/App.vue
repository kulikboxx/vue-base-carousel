<template>
  <base-carousel
    v-bind="{
      breakpoints,
      items,
      itemsPerView: 1,
      spaceBetween: 10,
    }"
  >
    <template #baseCarouselItem="{ item }">
      <div class="card">
        <p>
          <span>{{ item.id }})</span> {{ item.brand }}
        </p>
        <p>{{ item.price }}$</p>
      </div>
    </template>

    <template #baseCarouselPrev>
      {{ '<<<' }}
    </template>

    <template #baseCarouselNext>
      {{ '>>>' }}
    </template>
  </base-carousel>
</template>

<script setup lang="ts">
import BaseCarousel from './components/BaseCarousel.vue';

const items = [
  { id: 1, brand: 'Apple', price: 500 },
  { id: 2, brand: 'Samsung', price: 250 },
  { id: 3, brand: 'Nokia', price: 180 },
  { id: 4, brand: 'Sony', price: 360 },
  { id: 5, brand: 'LG', price: 260 },
];

const breakpoints = {
  250: { itemsPerView: 1 },
  450: { itemsPerView: 2 },
  680: { itemsPerView: 3 },
  850: { itemsPerView: 4 },
  1000: { itemsPerView: 5 },
};
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html {
  font-size: 62.5%;
  color-scheme: dark;
}

body {
  padding: 5rem 0;
  font-size: 1.6rem;
  font-family: Arial, sans-serif;
}

.card {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 20px;
  height: 100%;
  font-size: 3rem;
  background-color: #555;
}
</style>
