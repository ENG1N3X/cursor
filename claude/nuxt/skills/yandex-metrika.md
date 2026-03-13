---
name: yandex-metrika
description: "Яндекс.Метрика в Nuxt: модуль nuxt-yandex-metrika, ручной client-плагин, reachGoal, e-commerce, типизация window.ym. Используй при интеграции аналитики."
---

# Яндекс.Метрика в Nuxt

## Модуль: nuxt-yandex-metrika

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['nuxt-yandex-metrika'],
  yandexMetrika: {
    id: 'XXXXXXXX',
    debug: process.env.NODE_ENV !== 'production',
    delay: 0,
    cdn: false,
    verification: null,
    options: {
      webvisor: true,
      clickmap: true,
      trackLinks: true,
      accurateTrackBounce: true,
      trackHash: true,
    },
  },
})
```

## Отправка целей

```vue
<script setup lang="ts">
const { reachGoal } = useYandexMetrika()

function onSubmitForm() {
  reachGoal('form_submit', { form_name: 'contact' })
}

function onAddToCart(productId: string) {
  reachGoal('add_to_cart', { product_id: productId })
}
</script>
```

## Ручная интеграция (client plugin)

```ts
// app/plugins/yandex-metrika.client.ts
export default defineNuxtPlugin(() => {
  const config = useRuntimeConfig()
  const id = config.public.yandexMetrikaId

  if (!id || process.dev) return

  useHead({
    script: [
      {
        innerHTML: `
          (function(m,e,t,r,i,k,a){m[i]=m[i]||function(){(m[i].a=m[i].a||[]).push(arguments)};
          m[i].l=1*new Date();
          for(var j=0;j<document.scripts.length;j++){if(document.scripts[j].src===r)return;}
          k=e.createElement(t),a=e.getElementsByTagName(t)[0],k.async=1,k.src=r,a.parentNode.insertBefore(k,a)})
          (window, document, "script", "https://mc.yandex.ru/metrika/tag.js", "ym");
          ym(${id}, "init", {
            clickmap: true,
            trackLinks: true,
            accurateTrackBounce: true,
            webvisor: true,
            trackHash: true
          });
        `,
        type: 'text/javascript',
      },
    ],
    noscript: [
      {
        innerHTML: `<div><img src="https://mc.yandex.ru/watch/${id}" style="position:absolute; left:-9999px;" alt="" /></div>`,
      },
    ],
  })

  const router = useRouter()
  router.afterEach((to) => {
    if (window.ym) {
      window.ym(Number(id), 'hit', to.fullPath)
    }
  })

  return {
    provide: {
      metrikaGoal: (target: string, params?: Record<string, unknown>) => {
        if (window.ym) {
          window.ym(Number(id), 'reachGoal', target, params)
        }
      },
    },
  }
})
```

## Типизация window.ym

```ts
// shared/types/yandex-metrika.d.ts
declare global {
  interface Window {
    ym?: (
      counterId: number,
      action: 'init' | 'hit' | 'reachGoal' | 'params' | 'userParams',
      ...args: unknown[]
    ) => void
  }
}
export {}
```

## E-commerce

```ts
// composables/useEcommerce.ts
export function useEcommerceMetrika() {
  function pushPurchase(order: {
    id: string
    products: Array<{ id: string; name: string; price: number; quantity: number }>
    revenue: number
  }) {
    if (!window.dataLayer) window.dataLayer = []
    window.dataLayer.push({
      ecommerce: {
        purchase: {
          actionField: { id: order.id, revenue: order.revenue },
          products: order.products,
        },
      },
    })
  }

  return { pushPurchase }
}
```

## Частые ошибки

- ID счётчика захардкожен → используй env-переменную.
- `reachGoal` без проверки `window.ym` → crash если скрипт не загрузился.
- Нет типизации `window.ym` → TS-ошибки.
- SPA-переходы не трекаются → нужен `router.afterEach` с `ym('hit')`.
- Webvisor включён на dev/staging → шум в данных.
- Нет noscript fallback → потеря данных при заблокированном JS.
