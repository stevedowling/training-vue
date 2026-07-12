# 2 · Dynamic routes - URLs with parameters

> **You'll learn:** how one page component serves many URLs - `/products/1`, `/products/2`, `/products/anything` - and reads the parameter to know what to show.

## Why this matters

Real apps are full of *detail pages*: this product, that user, this order. You can't add a route per product - the products come from data, and next week there are forty more. Dynamic routes crack the pattern: one route, one component, a parameter that says which one. It's also half of the list → detail flow that basically every app on the internet is made of.

## The big picture

A `:parameter` in the path matches anything in that segment:

```js
// router/index.js
{ path: '/products/:id', component: ProductView }
```

| URL | Matches? | `route.params` |
|---|---|---|
| `/products/1` | ✅ | `{ id: '1' }` |
| `/products/42` | ✅ | `{ id: '42' }` |
| `/products/coffee` | ✅ | `{ id: 'coffee' }` |
| `/products` | ❌ | (no id segment - different route) |

The component reads the parameter with `useRoute()`:

```vue
<!-- src/views/ProductView.vue -->
<script setup>
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { products } from '../data/products'

const route = useRoute()
const product = computed(() =>
  products.find(p => p.id === Number(route.params.id))
)
</script>

<template>
  <div v-if="product">
    <h1>{{ product.emoji }} {{ product.name }}</h1>
    <p>${{ product.price }}</p>
  </div>
  <p v-else>No such product 🤷</p>
</template>
```

Two details doing quiet heavy lifting: params are **always strings** (hence `Number(...)` before comparing to numeric ids - the router's cousin of `.number` from Module 4), and the lookup is a **computed** so the page follows the URL reactively.

## The list → detail pattern

Dynamic params meet `v-for` and the shape of every catalogue, inbox, and feed appears:

```vue
<!-- ShopView.vue - each row now links to its detail page -->
<RouterLink v-for="p in products" :key="p.id" :to="`/products/${p.id}`">
  {{ p.emoji }} {{ p.name }}
</RouterLink>
```

List page links carry the id in the URL; detail page reads it back out. The URL *is* the communication channel - which is precisely what makes detail pages bookmarkable and sharable. (Notice `:to` with a template literal - the colon means expression, Module 3 forever.)

One structural consequence worth adopting now: the product array moves out of any single view into a shared module (`src/data/products.js` exporting the array) so both list and detail import the same data. A taste of Module 6, where shared *reactive* state gets the same treatment.

## When the URL changes but the component doesn't

Subtle and important: navigating from `/products/1` to `/products/2` **reuses the same component instance** - the router sees no reason to tear down and rebuild. Your refs keep their values; nothing "restarts".

With the computed-from-route pattern above, everything just works - the computed re-derives when `route.params` changes. The trap is code that reads the param *once* (in a plain `const` at setup, or a fetch fired only at component creation - Module 7 foreshadowing):

```js
const id = route.params.id                       // ❌ frozen at first visit
const id = computed(() => route.params.id)      // ✅ follows the URL
```

> [!TIP]
> House rule that avoids the whole trap family: **anything derived from `route.params` or `route.query` is a computed** (or a watched source). The URL is reactive state like any other - treat it with Module 2 manners.

<details>
<summary>🔍 Deep dive: route props, queries, and multiple params</summary>

Three neighbours of the core idea, all cheap to know:

**`props: true`** - add it to the route record and the router passes params as *props* (`{ path: '/products/:id', component: ProductView, props: true }` → the view declares `defineProps({ id: String })`). The view stops knowing about the router entirely - testable with plain props, reusable outside routing. Many teams' default style.

**Query strings** - `/shop?sort=price&max=10` lives in `route.query` (`{ sort: 'price', max: '10' }`), no route config needed. Convention: params for *identity* (which thing), query for *options* (how to show it) - filters, sorts, page numbers.

**Multiple params** - `/orders/:orderId/items/:itemId` works exactly as you'd guess; both land in `route.params`. Deeply nested UIs eventually meet the router's `children` option for literal nested layouts - the docs' Nested Routes page is where to look the day a page needs pages inside it.

</details>

## 🛠️ Try it - product pages for the Snack Cart

The shop gets a catalogue:

1. Move the products array to `src/data/products.js` (plain module, `export const products = [...]`). Give each product a `blurb` field - a sentence of loving snack prose. Update ShopView's import.
2. Add the dynamic route and build `ProductView.vue`: emoji big, name, blurb, price, and a RouterLink back to the shop. Handle the not-found case with `v-else` for now (lesson 3 upgrades it to a real 404).
3. Make each ProductRow's name a RouterLink to its detail page (the row already has the id - Module 3's props deliver again).
4. Prove the reactivity rule: put "next product →" / "← previous" links *on the detail page* (`/products/2` links to 3 and 1 - clamp or wrap at the ends, your call). Click through the whole catalogue without leaving the component. If the page follows, your computeds are right; if it freezes, you've met the reuse trap in person - fix and appreciate.
5. Bookmark test: copy a product URL, open it in a new tab. It works - that's the whole point of putting identity in the URL.

<details>
<summary>💡 Hint - next/previous without drama</summary>

```js
const idx = computed(() => products.findIndex(p => p.id === Number(route.params.id)))
const next = computed(() => products[idx.value + 1] ?? null)
const prev = computed(() => products[idx.value - 1] ?? null)
```

Then `<RouterLink v-if="next" :to="`/products/${next.id}`">` - computeds chaining off the route param, everything reactive end to end.

</details>

<details>
<summary>✅ Solution - the moving parts</summary>

```js
// src/data/products.js
export const products = [
  { id: 1, name: 'Coffee', price: 5, emoji: '☕', blurb: 'The reason mornings are survivable.' },
  { id: 2, name: 'Cookie', price: 3, emoji: '🍪', blurb: 'Baked. Never compiled.' },
  { id: 3, name: 'Toastie', price: 8, emoji: '🥪', blurb: 'Cheese with structural integrity.' },
]
```

```js
// router/index.js - the new record
{ path: '/products/:id', component: ProductView }
```

ProductView: the lesson's computed lookup, plus the hint's next/prev. ShopView rows: `<RouterLink :to="`/products/${p.id}`">{{ p.name }}</RouterLink>` inside the row template (or wrap the name slot - designer's choice).

</details>

## ✋ Checkpoint

1. `{ path: '/users/:username' }` - which of these URLs match, and what's in params: `/users/grace`, `/users/42`, `/users`, `/users/grace/posts`?
2. A detail page shows product 1 forever, even after clicking a link to product 2 (the URL changed!). Name the trap and the one-line class of fix.
3. `route.params.id === product.id` is false even though the page shows the right URL and `product.id` is `3`. Why?
4. Params or query: (a) which article to read, (b) dark-mode preview flag, (c) search results page number, (d) which user's profile?

<details>
<summary>Answers</summary>

1. `/users/grace` → `{ username: 'grace' }` and `/users/42` → `{ username: '42' }`. The other two don't match - missing segment and extra segment respectively.
2. Component reuse: the param was read once instead of reactively. Fix: derive via computed (or watch the param - Module 7 does exactly that for refetching).
3. Params are strings: `'3' === 3` is false. `Number(route.params.id)` before comparing.
4. (a) param - identity. (b) query - an option. (c) query - an option. (d) param - identity.

</details>

## 📚 Further reading

- [Dynamic Route Matching - Vue Router docs](https://router.vuejs.org/guide/essentials/dynamic-matching.html) - including sensitivity to the reuse behaviour
- [Passing Props to Route Components](https://router.vuejs.org/guide/essentials/passing-props.html) - the `props: true` style from the deep dive

---

⬅️ [Previous: Pages and links](./01-pages-and-links.md) · 🏠 [Course home](../README.md) · ➡️ [Next: Navigation in code + 404s](./03-navigation-in-code-and-404s.md)
