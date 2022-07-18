<template>
    <div
        class="base-carousel"
        ref="baseCarouselRef"
        @touchstart="onTouch($event)"
        @touchend="onTouch($event)"
        @transitionend="checkShiftX()"
    >
        <div class="base-carousel__line">
            <ul class="base-carousel__wrapper">
                <li v-for="(item, index) in transformedItems" :key="index" class="base-carousel__item">
                    <slot v-bind="{ item }" name="baseCarouselItem" />
                </li>
            </ul>
        </div>

        <div class="base-carousel__actions">
            <div class="base-carousel__prev" @click="swipeItem(1)">
                <slot name="baseCarouselPrev" />
            </div>
            <div class="base-carousel__next" @click="swipeItem(-1)">
                <slot name="baseCarouselNext" />
            </div>
        </div>
    </div>
</template>

<script lang="ts" setup>
import { computed, ref, onMounted, onUnmounted } from 'vue';

interface Props {
    duration?: number;
    items: Array<Record<PropertyKey, unknown>>;
    itemsPerView?: number;
    spaceBetween?: number;
    timing?: string;
}

const props = withDefaults(defineProps<Props>(), {
    duration: 300,
    itemsPerView: 1,
    spaceBetween: 0,
    timing: 'ease-in-out',
});

const baseCarouselRef = ref<HTMLDivElement>();

const itemSize = ref('');
const itemsGap = ref(props.spaceBetween + 'px');
const lineShift = ref('');
const shiftX = ref('');
const transition = ref('');

const currentItem = ref(0);
const touchStart = ref(0);

const transformedItems = computed(() => [...props.items, ...props.items, ...props.items]);

const resizeObserver = new ResizeObserver((entries) => {
    entries.forEach((entry) => {
        const carouselSize = entry.contentBoxSize[0].inlineSize;
        const whiteSpace = props.spaceBetween * (props.itemsPerView - 1);
        const elemSize = (carouselSize - whiteSpace) / props.itemsPerView;

        itemSize.value = elemSize + 'px';
        lineShift.value = -(elemSize + props.spaceBetween) * props.items.length + 'px';

        manageTransition(swipeItem);
    });
});

function swipeItem(num: number = 0) {
    const number = (currentItem.value += num);
    shiftX.value = number * parseFloat(itemSize.value) + number * props.spaceBetween + 'px';
}

function switchTransition(isTransition: boolean = true) {
    transition.value = isTransition ? `transform ${props.duration}ms ${props.timing}` : 'none';
}

function manageTransition(callback: Function) {
    switchTransition(false);
    callback();
    setTimeout(() => switchTransition(true));
}

function checkShiftX() {
    const itemWidthWithWhiteSpace = parseFloat(itemSize.value) + props.spaceBetween;
    const startX = parseFloat(shiftX.value) <= -(itemWidthWithWhiteSpace * props.items.length);
    const endX = parseFloat(shiftX.value) >= itemWidthWithWhiteSpace * props.itemsPerView;

    if (startX || endX) {
        manageTransition(() => {
            shiftX.value = startX ? '0px' : `${-(props.items.length - props.itemsPerView) * itemWidthWithWhiteSpace}px`;
            currentItem.value = startX ? 0 : -(props.items.length - props.itemsPerView);
        });
    }
}

function onTouch(e: TouchEvent) {
    let touchEnd = 0;

    if (e.type === 'touchstart') {
        touchStart.value = e.touches[0].clientX;
    } else {
        touchEnd = e.changedTouches[0].clientX;
    }

    if (touchStart.value && touchEnd) {
        swipeItem(touchStart.value - touchEnd < 0 ? 1 : -1);
    }
}

function onKeyDown(e: KeyboardEvent) {
    if (e.key === 'ArrowLeft') swipeItem(1);
    if (e.key === 'ArrowRight') swipeItem(-1);
}

onMounted(() => {
    switchTransition(true);

    if (baseCarouselRef.value) {
        resizeObserver.observe(baseCarouselRef.value);
    }

    document.addEventListener('keydown', onKeyDown);
});

onUnmounted(() => {
    if (baseCarouselRef.value) {
        resizeObserver.unobserve(baseCarouselRef.value);
    }

    document.removeEventListener('keydown', onKeyDown);
});
</script>

<style lang="scss">
.base-carousel {
    position: relative;
    margin-inline: auto;
    padding: 1rem;
    width: 100%;
    max-width: 1200px;
    background-color: #444;
    user-select: none;
    overflow: hidden;
    z-index: 0;

    &__line,
    &__wrapper,
    &__item {
        height: 100%;
    }

    &__line {
        transform: translateX(v-bind(lineShift));
    }

    &__wrapper {
        display: flex;
        align-items: stretch;
        width: max-content;
        gap: v-bind(itemsGap);
        transition: v-bind(transition);
        transform: translateX(v-bind(shiftX));
        list-style: none;
    }

    &__item {
        width: v-bind(itemSize);
        height: 400px;
    }

    &__actions {
        position: absolute;
        top: 50%;
        left: 0;
        right: 0;
        transform: translateY(-50%);
        display: flex;
        justify-content: space-between;
    }

    &__prev,
    &__next {
        cursor: pointer;
    }
}
</style>
