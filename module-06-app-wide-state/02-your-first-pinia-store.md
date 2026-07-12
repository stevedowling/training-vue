# 2 · Your first Pinia store

> **You'll learn:** how to create a Pinia store and use it from any component - shared state with a proper home, plus the one destructuring gotcha that catches everyone.

## Why this matters

Lesson 1 built the case and even a prototype (the loose-ref module). Pinia is that idea with a uniform: a standard shape every Vue developer recognises, first-class devtools, and a name for each part. It's the official state library, the capstone uses it, and - good news - if you followed Module 2, you already know 90% of its API.

## The big picture

Install and register (once per app):

```bash
npm install pinia
```

```js
// main.js
import { createPinia } from 'pinia'
createApp(App).use(createPinia()).use(router).mount('#app')
```

A store is a file in `src/stores/`, and here's the punchline of the whole module - *it's Module 2 inside a wrapper*:

```js
// src/stores/cart.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useCartStore = defineStore('cart', () => {
  const qty = ref({})                                        // state = refs

  const itemCount = computed(() =>                           // getters = computeds
    Object.values(qty.value).reduce((s, n) => s + n, 0)
  )

  function add(id) { qty.value[id] = (qty.value[id] ?? 0) + 1 }   // actions = functions
  function remove(id) { if ((qty.value[id] ?? 0) > 0) qty.value[id]-- }

  return { qty, itemCount, add, remove }                     // expose what components may touch
})
```

`defineStore('cart', setupFunction)` - a unique id (devtools uses it) and a function that looks exactly like a component's `<script setup>`: refs, computeds, functions, returned. No new reactivity to learn; the skill transfers whole.

## Using it anywhere

```vue
<!-- CartBadge.vue - five components from the shop, zero drilling -->
<script setup>
import { useCartStore } from '../stores/cart'
const cart = useCartStore()
</script>

<template>
  <span>🛒 {{ cart.itemCount }}</span>
</template>
```

```vue
<!-- ProductRow's parent, ShopView.vue -->
<script setup>
import { useCartStore } from '../stores/cart'
const cart = useCartStore()
// template: @add="cart.add" @remove="cart.remove" :qty="cart.qty[p.id] ?? 0"
</script>
```

Call the `use...Store()` function in any component; every caller gets the **same instance** (first call creates it, the rest join). On the store object, refs and computeds arrive *pre-unwrapped* - `cart.itemCount`, no `.value` - and actions are just methods. Change state in one component, every subscriber re-renders: it's Module 2 reactivity with an address anyone can reach.

## The gotcha: destructuring loses the magic

Module 2 lesson 2 warned that destructuring reactive objects hands you dead values. Stores are reactive objects:

```js
const { itemCount } = useCartStore()     // ❌ frozen at this moment, forever
```

```js
import { storeToRefs } from 'pinia'
const { itemCount, qty } = storeToRefs(useCartStore())   // ✅ live refs (.value in script!)
const { add, remove } = useCartStore()                    // ✅ actions are plain functions - safe
```

The rule: **state and getters through `storeToRefs`, actions straight off the store** - or skip destructuring entirely and write `cart.itemCount` everywhere, which plenty of teams prefer for the free namespacing (readers always know where a value lives).

> [!WARNING]
> The symptom of the bad destructure is the cruellest kind: no error, no warning, a badge that shows 0 forever while the store hums along correctly. When shared state "doesn't update", check for a naked store destructure before anything else.

<details>
<summary>🔍 Deep dive: the options syntax, and where the loose ref fell short</summary>

Two things you'll meet in the wild. First, Pinia's *other* dialect - options stores:

```js
export const useCartStore = defineStore('cart', {
  state: () => ({ qty: {} }),
  getters: { itemCount: (state) => Object.values(state.qty).reduce((s, n) => s + n, 0) },
  actions: { add(id) { this.qty[id] = (this.qty[id] ?? 0) + 1 } },
})
```

Same store, Options-API flavour (Module 1's note about the two styles, echoing). Tutorials use it constantly; translate on sight: `state`→refs, `getters`→computeds, `actions`→functions with `this`. The course writes setup stores because the skills are literally Module 2's.

Second: open Vue devtools → Pinia tab with your store running. Live state, a timeline of every action with payloads, and editable values - debugging shared state stops being print-statement archaeology, which is a real chunk of why the uniform beats the loose ref. (The rest: SSR safety, plugin ecosystem, and hot-module-reload that survives edits.)

</details>

## 🛠️ Try it - the store-powered Snack Shop

Convert lesson 1's cart to Pinia and cash every cheque this module has written:

1. Install Pinia, register it, create `src/stores/cart.js` exactly as the lesson has it - then add a `total` getter (it needs the products list: import the array from `src/data/products.js` right into the store; stores may import data modules freely).
2. Rewire: ShopView uses `cart.add`/`cart.remove`/`cart.qty`; CartBadge reads `cart.itemCount`; App.vue gives back every drilled prop from lesson 1 and returns to being a frame. Delete `src/state/cart.js` if you built it - the prototype is honourably discharged.
3. The route-death exorcism: add items, wander to Home, Pizza, a product page, come back. Cart intact. Then upgrade Module 5's checkout: ThanksView reads `itemCount`/`total` *from the store* and the query-string smuggling gets deleted with prejudice. Add a `clear()` action and call it after "ordering".
4. Meet the gotcha on purpose: in CartBadge, destructure `const { itemCount } = useCartStore()`, click + in the shop, watch the badge freeze while the shop total moves. Fix with `storeToRefs`. Two minutes now saves two hours someday.
5. Devtools: open the Pinia tab, click + a few times, watch actions land in the timeline. Edit `qty` in devtools directly and watch the UI obey.

<details>
<summary>💡 Hint - the total getter</summary>

```js
import { products } from '../data/products'

const total = computed(() =>
  products.reduce((sum, p) => sum + p.price * (qty.value[p.id] ?? 0), 0)
)
```

Return it alongside the rest. (Products are static data, not state - importing them into the store is clean; *reactive* product data comes in Module 7's world.)

</details>

<details>
<summary>✅ Solution - the finished store</summary>

```js
// src/stores/cart.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import { products } from '../data/products'

export const useCartStore = defineStore('cart', () => {
  const qty = ref({})

  const itemCount = computed(() => Object.values(qty.value).reduce((s, n) => s + n, 0))
  const total = computed(() =>
    products.reduce((sum, p) => sum + p.price * (qty.value[p.id] ?? 0), 0)
  )

  function add(id) { qty.value[id] = (qty.value[id] ?? 0) + 1 }
  function remove(id) { if ((qty.value[id] ?? 0) > 0) qty.value[id]-- }
  function clear() { qty.value = {} }

  return { qty, itemCount, total, add, remove, clear }
})
```

ThanksView: `const cart = useCartStore()` - show `cart.itemCount` / `cart.total`, call `cart.clear()` on a "Done" button (or after showing the summary; design taste). The checkout push shrinks to `router.push('/thanks')` - no cargo.

</details>

## ✋ Checkpoint

1. Map the vocabulary: state, getters, actions - what does each correspond to in Module 2 terms, in a setup store?
2. Two components both call `useCartStore()`. How many stores exist, and what does the *first* call do that the second doesn't?
3. Predict: `const { total, clear } = useCartStore()` - which of the two destructured names is broken, which is fine, and what's the repair kit called?
4. Lesson 1 shipped cart totals to ThanksView in a query string. The store made that vanish - explain in one sentence *why* a store survives the navigation when ShopView's refs didn't.

<details>
<summary>Answers</summary>

1. State = refs, getters = computeds, actions = plain functions. (The whole exam of Module 2, disguised as vocabulary.)
2. One - stores are singletons per id. The first call runs the setup function and creates it; later calls return the existing instance.
3. `total` is dead (state/getter through a naked destructure); `clear` is fine (actions are plain functions). Repair kit: `storeToRefs` for the former.
4. The store lives outside the component tree - navigation unmounts *components*, and the store isn't one; it's a module-level object that persists for the life of the app.

</details>

## 📚 Further reading

- [Pinia: Core concepts](https://pinia.vuejs.org/core-concepts/) - defineStore in both dialects, official
- [storeToRefs](https://pinia.vuejs.org/api/modules/pinia.html#storetorefs) - the gotcha's official documentation

---

⬅️ [Previous: The prop-drilling problem](./01-the-prop-drilling-problem.md) · 🏠 [Course home](../README.md) · ➡️ [Next: What goes in the store](./03-what-goes-in-the-store.md)
