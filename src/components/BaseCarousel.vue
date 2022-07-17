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
                <li v-for="(item, index) in computedItems" :key="index" class="base-carousel__item">
                    <slot v-bind="{ item }" name="baseCarouselItem" />
                </li>
            </ul>
        </div>

        <button class="base-carousel__prev" @click="swipeItem(1)">
            <slot name="baseCarouselPrev" />
        </button>
        <button class="base-carousel__next" @click="swipeItem(-1)">
            <slot name="baseCarouselNext" />
        </button>
    </div>
</template>

<script lang="ts" setup>
import { computed, reactive, ref, onMounted, onUnmounted } from 'vue';

interface Props {
    duration?: number;
    items: Array<any>;
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
const carouselStyles = reactive({
    offsetWidth: 0,
    padding: 0,
    whiteSpace: 0,
});

const currentItem = ref(0);
const shiftX = ref(0);
const touchStart = ref(0);
const transition = ref('');

function getCarouselStyles() {
    if (baseCarouselRef.value) {
        const carousel = baseCarouselRef.value;

        const paddingLeft = parseFloat(window.getComputedStyle(carousel).paddingLeft);
        const paddingRight = parseFloat(window.getComputedStyle(carousel).paddingRight);

        carouselStyles.offsetWidth = parseFloat(window.getComputedStyle(carousel).width);
        carouselStyles.padding = paddingLeft + paddingRight;
        carouselStyles.whiteSpace = props.spaceBetween * (props.itemsPerView - 1);
    }
}

const computedItems = computed(() => [...props.items, ...props.items, ...props.items]);
const computedItemGap = computed(() => props.spaceBetween + 'px');
const computedShiftX = computed(() => shiftX.value + 'px');

const computedItemWidth = computed(
    () => (carouselStyles.offsetWidth - carouselStyles.padding - carouselStyles.whiteSpace) / props.itemsPerView + 'px'
);

const computedLineShift = computed(
    () => -(parseFloat(computedItemWidth.value) + props.spaceBetween) * props.items.length + 'px'
);

function swipeItem(num: number) {
    const number = (currentItem.value += num);
    shiftX.value = number * parseFloat(computedItemWidth.value) + number * props.spaceBetween;
}

function toggleTransition(isTransition: boolean = true) {
    transition.value = isTransition ? `transform ${props.duration}ms ${props.timing}` : 'none';
}

function checkShiftX() {
    const itemWidthWithWhiteSpace = parseFloat(computedItemWidth.value) + props.spaceBetween;
    const startShiftX = shiftX.value <= -(itemWidthWithWhiteSpace * props.items.length);
    const endShiftX = shiftX.value >= itemWidthWithWhiteSpace * props.itemsPerView;

    if (startShiftX || endShiftX) {
        toggleTransition(false);

        shiftX.value = startShiftX ? 0 : -(props.items.length - props.itemsPerView) * itemWidthWithWhiteSpace;
        currentItem.value = startShiftX ? 0 : -(props.items.length - props.itemsPerView);

        setTimeout(() => toggleTransition(true));
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

function toggleTriggers(isMounted: boolean = true) {
    if (isMounted) {
        window.addEventListener('resize', initCarousel);
        document.addEventListener('keydown', onKeyDown);
        return;
    }

    window.removeEventListener('resize', initCarousel);
    document.removeEventListener('keydown', onKeyDown);
}

function initCarousel() {
    getCarouselStyles();
    toggleTransition(true);
    swipeItem(0);
    toggleTriggers();
}

onMounted(() => initCarousel());
onUnmounted(() => toggleTriggers(false));
</script>

<style lang="scss">
.base-carousel {
    position: relative;
    margin-inline: auto;
    width: 100%;
    max-width: 1200px;
    height: 400px;
    user-select: none;
    overflow: hidden;
    z-index: 0;

    &__line,
    &__wrapper,
    &__item {
        height: 100%;
    }

    &__line {
        transform: translateX(v-bind(computedLineShift));
    }

    &__wrapper {
        display: flex;
        align-items: stretch;
        width: max-content;
        gap: v-bind(computedItemGap);
        transition: v-bind(transition);
        transform: translateX(v-bind(computedShiftX));
        list-style: none;
    }

    &__item {
        width: v-bind(computedItemWidth);
    }

    &__prev,
    &__next {
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
        display: inline-flex;
        justify-content: center;
        align-items: center;
        background: none;
        border: none;
        cursor: pointer;

        &:focus {
            outline: none;
        }
    }

    &__prev {
        left: 0;
    }

    &__next {
        right: 0;
    }
}
</style>
