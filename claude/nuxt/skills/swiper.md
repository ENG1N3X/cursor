---
name: swiper-nuxt
description: "Swiper.js в Nuxt: nuxt-swiper (web components), swiper/vue, SSR/hydration, ClientOnly. Используй при создании слайдеров, каруселей, галерей."
---

# Swiper.js в Nuxt

## Подход 1: nuxt-swiper (рекомендуемый)

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['nuxt-swiper'],
})
```

Swiper v11+ использует Web Components:

```vue
<script setup lang="ts">
const containerRef = ref<HTMLElement | null>(null)
const swiper = useSwiper(containerRef, {
  loop: true,
  autoplay: { delay: 5000, disableOnInteraction: false },
  pagination: { clickable: true },
  slidesPerView: 1,
  spaceBetween: 24,
  breakpoints: {
    640: { slidesPerView: 2 },
    1024: { slidesPerView: 3 },
  },
})
</script>

<template>
  <ClientOnly>
    <swiper-container ref="containerRef" :init="false">
      <swiper-slide v-for="(item, idx) in items" :key="idx">
        <SlideContent :data="item" />
      </swiper-slide>
    </swiper-container>
  </ClientOnly>
</template>
```

**Критически важно:**
- `<ClientOnly>` обязателен — Web Components не работают в SSR.
- `:init="false"` + `useSwiper` для программного контроля.
- kebab-case: `<swiper-container>`, `<swiper-slide>`, НЕ PascalCase.
- Стилизация через CSS `::part()` или внутренний div.

## Подход 2: swiper/vue

```vue
<script setup lang="ts">
import { Swiper, SwiperSlide } from 'swiper/vue'
import { Navigation, Pagination, Autoplay } from 'swiper/modules'
import 'swiper/css'
import 'swiper/css/navigation'
import 'swiper/css/pagination'

const modules = [Navigation, Pagination, Autoplay]
</script>

<template>
  <Swiper
    :modules="modules"
    :slides-per-view="3"
    :space-between="20"
    navigation
    :pagination="{ clickable: true }"
    :autoplay="{ delay: 3000 }"
  >
    <SwiperSlide v-for="item in items" :key="item.id">
      <SlideContent :data="item" />
    </SwiperSlide>
  </Swiper>
</template>
```

**Когда какой:**
- `nuxt-swiper` — новые проекты, простая интеграция.
- `swiper/vue` — тонкий контроль, кастомные модули, legacy.

## Частые проблемы

- Hydration mismatch → `<ClientOnly>`.
- Слайды пустые → `v-if="items.length"` на контейнере.
- Autoplay умирает после свайпа → `disableOnInteraction: false`.
- Pagination не видна → импортируй `swiper/css/pagination`.
- SSR ошибки с `swiper/vue` → `build: { transpile: ['swiper'] }`.
