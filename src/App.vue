<template>
    <base-carousel v-bind="{ items: elements, itemsPerView: 4, spaceBetween: 10 }">
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

const elements = [
    { id: 1, brand: 'Apple', price: 500 },
    { id: 2, brand: 'Samsung', price: 250 },
    { id: 3, brand: 'Nokia', price: 180 },
    { id: 4, brand: 'Sony', price: 360 },
];
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
