---
name: tailwind-v4
description: "Tailwind CSS v4 в Nuxt: установка, CSS-first конфигурация через @theme, динамические классы, антипаттерны. Используй при работе со стилями, UI, адаптивностью."
---

# Tailwind CSS v4 в Nuxt

## Установка (два пути)

Путь 1 — через Vite-плагин (рекомендуемый для новых проектов):
```ts
// nuxt.config.ts
import tailwindcss from '@tailwindcss/vite'

export default defineNuxtConfig({
  css: ['~/assets/css/main.css'],
  vite: {
    plugins: [tailwindcss()],
  },
})
```

Путь 2 — через `@nuxtjs/tailwindcss` модуль v6+ (для миграции с v3):
```ts
export default defineNuxtConfig({
  modules: ['@nuxtjs/tailwindcss'],
})
```

**ВАЖНО: Если используется `@nuxt/ui` v3+, он уже включает Tailwind v4. НЕ добавляй `@nuxtjs/tailwindcss` отдельно — будет конфликт.**

## CSS-first конфигурация

```css
/* app/assets/css/main.css */
@import "tailwindcss";

@theme {
  --color-brand: #3b82f6;
  --color-brand-dark: #1d4ed8;
  --font-display: "Inter", sans-serif;
}
```

## Ключевые изменения v4 vs v3

- Нет `tailwind.config.js` — конфигурация через CSS `@theme`.
- Нет `content` путей — Tailwind v4 сканирует автоматически.
- Движок Oxide (Rust) — билды до 5x быстрее.
- Динамические классы: используй CSS-переменные вместо конкатенации строк.

## Динамические классы

```vue
<!-- НЕПРАВИЛЬНО: Tailwind v4 не увидит -->
<div :class="`bg-${color}-500`">

<!-- ПРАВИЛЬНО: CSS-переменные -->
<div :style="{ '--tw-bg': color }" class="bg-[var(--tw-bg)]">

<!-- ПРАВИЛЬНО: полные классы -->
<div :class="{ 'bg-red-500': isError, 'bg-green-500': isSuccess }">
```

## Антипаттерны

- Конкатенация строк для классов → Tailwind v4 не сканирует runtime-значения.
- `tailwind.config.js` в новом проекте → используй `@theme` в CSS.
- `@nuxtjs/tailwindcss` + `@nuxt/ui` одновременно → конфликт, убери модуль.
- `@apply` для одноразовых классов → bloat, используй утилиты напрямую.
- Конфликтующие утилиты на одном элементе (`p-4 p-8`).
- Inline styles вместо Tailwind-классов без причины.
- Захардкоженные px вместо Tailwind spacing.
