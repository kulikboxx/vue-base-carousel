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
        <li
          v-for="(item, index) in computedItems"
          :key="index"
          class="base-carousel__item"
        >
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
import { computed, reactive, ref, onMounted, onBeforeUnmount } from 'vue';

interface Props {
  breakpoints?: Record<PropertyKey, Record<string, number>>;
  items: Array<Record<PropertyKey, unknown>>;
  itemsPerView?: number;
  loopDuration?: number;
  spaceBetween?: number;
  transitionDuration?: number;
  timing?: string;
}

const props = withDefaults(defineProps<Props>(), {
  breakpoints: () => ({ 200: { itemsPerView: 1 } }),
  itemsPerView: 1,
  loopDuration: 6000,
  spaceBetween: 0,
  transitionDuration: 1200,
  timing: 'ease',
});

const baseCarouselRef = ref<HTMLDivElement>();
const loopTimer = ref(0);

const carousel = reactive({
  currentItem: 0,
  itemWidth: '',
  itemsGap: `${props.spaceBetween}px`,
  lineShift: '',
  minItemWidth: 200,
  shiftX: '',
  transition: '',
  touchStart: 0,
  whiteSpaces: 0,
  visibleItems: 1,
});

const computedItems = computed(() => [
  ...props.items,
  ...props.items,
  ...props.items,
]);

const carouselObserver = new ResizeObserver((entries) => {
  entries.forEach((entry) => {
    computeItemWidth();
    computeWhiteSpaces();
    setItemWidth(entry.contentBoxSize[0].inlineSize);
    setLineShift();
    manageTransition(swipeItem);
  });
});

const computedBreakpoints = computed(() => {
  const breakpoints = [];

  for (const key in props.breakpoints) {
    breakpoints.push({
      breakpoint: +key,
      visibleItems: props.breakpoints[key].itemsPerView,
    });
  }

  return breakpoints;
});

function computeWhiteSpaces() {
  carousel.whiteSpaces =
    parseFloat(carousel.itemsGap) * (carousel.visibleItems - 1);
}

function computeItemWidth() {
  const breakpointsArray = computedBreakpoints.value;
  const screenWidth = window.innerWidth;
  const lastBreakpoint = breakpointsArray[breakpointsArray.length - 1];

  for (let i = 0; i < breakpointsArray.length; i++) {
    if (
      screenWidth < lastBreakpoint.breakpoint &&
      screenWidth >= breakpointsArray[i].breakpoint &&
      screenWidth <= breakpointsArray[i + 1].breakpoint
    ) {
      carousel.visibleItems = breakpointsArray[i].visibleItems;
    } else if (screenWidth >= lastBreakpoint.breakpoint) {
      carousel.visibleItems = lastBreakpoint.visibleItems;
    }
  }
}

function setItemWidth(carouselWidth: number) {
  carousel.itemWidth =
    (carouselWidth - carousel.whiteSpaces) / carousel.visibleItems + 'px';
}

function setLineShift() {
  carousel.lineShift =
    -(parseFloat(carousel.itemWidth) + props.spaceBetween) *
      props.items.length +
    'px';
}

function swipeItem(num: number = 0) {
  updateLoopTimer(true);

  const number = (carousel.currentItem += num);
  carousel.shiftX =
    number * parseFloat(carousel.itemWidth) +
    number * props.spaceBetween +
    'px';
}

function switchTransition(isTransition: boolean = true) {
  carousel.transition = isTransition
    ? `transform ${props.transitionDuration}ms ${props.timing}`
    : 'none';
}

function manageTransition(callback: Function) {
  switchTransition(false);
  callback();
  setTimeout(() => switchTransition(true));
}

function checkShiftX() {
  const itemWidthWithWhiteSpace =
    parseFloat(carousel.itemWidth) + props.spaceBetween;
  const startX =
    parseFloat(carousel.shiftX) <=
    -(itemWidthWithWhiteSpace * props.items.length);
  const endX =
    parseFloat(carousel.shiftX) >= itemWidthWithWhiteSpace * props.itemsPerView;

  if (startX || endX) {
    manageTransition(() => {
      carousel.shiftX = startX
        ? '0px'
        : `${
            -(props.items.length - props.itemsPerView) * itemWidthWithWhiteSpace
          }px`;

      carousel.currentItem = startX
        ? 0
        : -(props.items.length - props.itemsPerView);
    });
  }
}

function updateLoopTimer(start?: boolean) {
  clearInterval(loopTimer.value);

  if (!start) return;

  loopTimer.value = setInterval(loopCarousel, props.loopDuration);
}

function onTouch(e: TouchEvent) {
  let touchEnd = 0;

  if (e.type === 'touchstart') {
    carousel.touchStart = e.touches[0].clientX;
  } else {
    touchEnd = e.changedTouches[0].clientX;
  }

  if (carousel.touchStart && touchEnd) {
    swipeItem(carousel.touchStart - touchEnd < 0 ? 1 : -1);
  }
}

function onKeyDown(e: KeyboardEvent) {
  if (e.key === 'ArrowLeft') swipeItem(1);
  if (e.key === 'ArrowRight') swipeItem(-1);
}

function loopCarousel() {
  swipeItem(-1);
}

onMounted(() => {
  switchTransition(true);
  updateLoopTimer(true);

  if (baseCarouselRef.value) {
    carouselObserver.observe(baseCarouselRef.value);
  }

  document.addEventListener('keydown', onKeyDown);
});

onBeforeUnmount(() => {
  if (baseCarouselRef.value) {
    carouselObserver.unobserve(baseCarouselRef.value);
  }

  updateLoopTimer(false);
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
    transform: translateX(v-bind('carousel.lineShift'));
  }

  &__wrapper {
    display: flex;
    align-items: stretch;
    width: max-content;
    gap: v-bind('carousel.itemsGap');
    transition: v-bind('carousel.transition');
    transform: translateX(v-bind('carousel.shiftX'));
    list-style: none;
  }

  &__item {
    width: v-bind('carousel.itemWidth');
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
