---
name: nuxt-patterns
description: "Паттерны Nuxt: data fetching, серверные API (Nitro), SEO, гибридный рендеринг, обработка ошибок, middleware. Используй при написании бизнес-логики, API, страниц."
---

# Паттерны Nuxt

## Data Fetching

```vue
<script setup lang="ts">
// GET — useFetch (SSR + клиент, кеш, дедупликация)
const { data: posts, status } = await useFetch('/api/posts', {
  query: { page: currentPage },
  watch: [currentPage],
})

// Мутации — $fetch (POST, PUT, DELETE)
async function createPost(body: CreatePostInput) {
  try {
    const post = await $fetch('/api/posts', {
      method: 'POST',
      body,
    })
    return post
  } catch (error) {
    if (error instanceof FetchError) {
      throw createError({
        statusCode: error.statusCode,
        statusMessage: error.statusMessage,
      })
    }
    throw error
  }
}
</script>
```

**Правило:** `useFetch` / `useAsyncData` — для GET. `$fetch` — для мутаций и клиентских запросов вне setup.

## Server API (Nitro)

```ts
// server/api/posts/index.get.ts
import { z } from 'zod'

const querySchema = z.object({
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(20),
})

export default defineEventHandler(async (event) => {
  const query = await getValidatedQuery(event, querySchema.parse)
  const { page, limit } = query
  const offset = (page - 1) * limit

  const [posts, total] = await Promise.all([
    db.select().from(postsTable).limit(limit).offset(offset),
    db.select({ count: count() }).from(postsTable),
  ])

  return {
    data: posts,
    meta: { page, limit, total: total[0].count, totalPages: Math.ceil(total[0].count / limit) },
  }
})
```

```ts
// server/api/posts/index.post.ts
import { createPostSchema } from '~~/shared/validators/post'

export default defineEventHandler(async (event) => {
  const session = await requireAuth(event)
  const body = await readValidatedBody(event, createPostSchema.parse)

  const post = await db.insert(postsTable).values({
    ...body,
    authorId: session.userId,
  }).returning()

  setResponseStatus(event, 201)
  return post[0]
})
```

## Middleware

```ts
// app/middleware/auth.ts
export default defineNuxtRouteMiddleware(async (to) => {
  const { loggedIn } = useUserSession()

  if (!loggedIn.value) {
    return navigateTo('/login', { redirectCode: 302, external: false })
  }
})
```

## SEO

```vue
<script setup lang="ts">
useSeoMeta({
  title: () => post.value?.title ?? 'Загрузка...',
  ogTitle: () => post.value?.title,
  description: () => post.value?.excerpt,
  ogDescription: () => post.value?.excerpt,
  ogImage: () => post.value?.coverImage,
  twitterCard: 'summary_large_image',
})

useHead({
  script: [
    {
      type: 'application/ld+json',
      innerHTML: JSON.stringify({
        '@context': 'https://schema.org',
        '@type': 'Article',
        headline: post.value?.title,
        image: post.value?.coverImage,
        datePublished: post.value?.createdAt,
        author: { '@type': 'Person', name: post.value?.author?.name },
      }),
    },
  ],
})
</script>
```

## Гибридный рендеринг

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  routeRules: {
    '/': { prerender: true },                     // SSG
    '/blog/**': { isr: 3600 },                    // ISR
    '/dashboard/**': { ssr: false },              // SPA
    '/api/**': { cors: true, headers: { 'cache-control': 'no-store' } },
    '/catalog/**': { swr: 600 },                  // Stale-While-Revalidate
  },
})
```

## Обработка ошибок

```vue
<!-- app/error.vue -->
<script setup lang="ts">
import type { NuxtError } from '#app'

const props = defineProps<{ error: NuxtError }>()
const handleError = () => clearError({ redirect: '/' })

const errorMessages: Record<number, string> = {
  404: 'Страница не найдена',
  403: 'Доступ запрещён',
  500: 'Внутренняя ошибка сервера',
}

const message = computed(() =>
  errorMessages[props.error.statusCode] ?? 'Произошла ошибка'
)
</script>

<template>
  <div class="min-h-screen flex items-center justify-center">
    <div class="text-center">
      <h1 class="text-6xl font-bold text-gray-800">{{ error.statusCode }}</h1>
      <p class="mt-4 text-lg text-gray-600">{{ message }}</p>
      <button class="mt-6 btn-primary" @click="handleError">На главную</button>
    </div>
  </div>
</template>
```
