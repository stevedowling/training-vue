# 3 · Composables - packaging the pattern for reuse

> **You'll learn:** how to extract reactive logic into a reusable function - building `useFetch`, wiring a search-as-you-type box with it, and meeting the pattern that powers the entire Vue ecosystem.

## Why this matters

Lesson 2's pattern is correct - and it's forty lines you'd re-type in every data-touching component forever. Components solved this for *UI* (write once, use everywhere); **composables** are the same move for *logic*. This is Vue's flagship reuse story: the ecosystem's biggest library is nothing but composables, every `use...` you've called (`useRoute`, `useCartStore`) is one, and today you write your own.

## The big picture

A composable is a function that uses reactivity and returns refs:

```js
// src/composables/useFetch.js
import { ref } from 'vue'

export function useFetch(url) {
  const data = ref(null)
  const error = ref(null)
  const loading = ref(false)

  async function load() {
    loading.value = true
    error.value = null
    try {
      const res = await fetch(url)
      if (!res.ok) throw new Error(`HTTP ${res.status}`)
      data.value = await res.json()
    } catch (e) {
      error.value = e.message
    } finally {
      loading.value = false
    }
  }

  load()
  return { data, error, loading, retry: load }
}
```

```vue
<!-- any component, forever after -->
<script setup>
import { useFetch } from '../composables/useFetch'
const { data, error, loading, retry } = useFetch('https://dummyjson.com/products?limit=12')
</script>
```

Lesson 2's entire ceremony: one line per component now. Look at what the function is made of - refs, the hardened fetch, a return. No component, no template, no new Vue feature: **a composable is just setup-code in a suitcase.** (And destructuring the return is safe - it hands back the refs themselves, `.value` and all, unlike the store objects of Module 6.)

## The conventions that make it a composable

- **Name starts with `use`** - `useFetch`, `useLocalStorage`, `useMousePosition`. It signals "call me inside setup, I create reactive state".
- **Lives in `src/composables/`**, one per file.
- **Returns refs in an object** so callers destructure what they need (and can rename: `const { data: products } = useFetch(...)` - handy the moment a component uses two fetches).
- **Each call is a fresh instance** - two components calling `useFetch` get independent state. Spot the contrast: *stores share, composables instantiate.* (Composable = "a new gadget each time you ask"; store = "the one whiteboard everyone reads.")

## Making it reactive: fetch-follows-ref

The version above fetches once. The upgrade that makes it *Vue-shaped*: accept a ref, and refetch whenever it changes:

```js
import { ref, watch, toValue } from 'vue'

export function useFetch(urlSource) {
  // ...same refs, same load(), but load reads:  fetch(toValue(urlSource))
  watch(() => toValue(urlSource), load, { immediate: true })
  return { data, error, loading, retry: load }
}
```

`toValue` unwraps whatever arrives - a plain string, a ref, or a getter function - so *all* of these work:

```js
useFetch('https://...')                                          // static: fetches once
useFetch(searchUrl)                                              // ref: refetches when it changes
useFetch(() => `https://.../search?q=${encodeURIComponent(q.value)}`)   // getter: same, computed-style
```

That last form plus a `v-model` is literally a live search box - the component's entire job becomes wiring input to URL:

```vue
<script setup>
const q = ref('')
const { data, loading, error } = useFetch(
  () => `https://dummyjson.com/products/search?q=${encodeURIComponent(q.value)}&limit=8`
)
</script>

<template>
  <input v-model="q" placeholder="Search products...">
  <!-- lesson 2's four lanes, reading the composable's refs -->
</template>
```

Type "pho" → the getter's value changes three times → the watcher refires → results follow the keystrokes. Modules 2, 4, and 7 in one wire.

> [!TIP]
> Extract a composable the same way you extract a component: **on the second copy-paste**, not speculatively. And the same judgment question as Module 6, inverted, decides between the two tools: shared state → store; repeated *stateful logic*, fresh per caller → composable.

<details>
<summary>🔍 Deep dive: debouncing, and the library that already wrote your composable</summary>

Search-as-you-type has two rough edges. One you've met - the stale-response race (lesson 2's deep dive; the id-guard belongs *inside* useFetch now, fixed once for every consumer - that's the suitcase paying off). The other is volume: "phone" fires five requests for one thought. The fix is **debouncing** - wait until typing pauses:

```js
// composables/useDebouncedRef.js - a composable that wraps a ref
import { ref, watch } from 'vue'
export function useDebouncedRef(source, ms = 300) {
  const debounced = ref(source.value)
  let timer
  watch(source, (v) => {
    clearTimeout(timer)
    timer = setTimeout(() => debounced.value = v, ms)
  })
  return debounced
}
```

`const slowQ = useDebouncedRef(q)` then build the URL from `slowQ` - composables composing, hence the name.

And the ecosystem punchline: **[VueUse](https://vueuse.org)** is 200+ production-grade composables - `useFetch` (with abort, refetch, and debounce built in), `useLocalStorage` (Module 2's watcher exercise, one line), `useDebounceFn`, `useDark` (Module 6's theme store, mostly)... You built yours to *understand* the pattern; in real projects, check VueUse before writing one. Reading its source is a masterclass in exactly what this lesson taught.

</details>

## 🛠️ Try it - useFetch and the live search

The module's finale:

1. Extract `src/composables/useFetch.js` - the reactive version with `toValue` + `watch` + `immediate`, the `res.ok` throw, and lesson 2's request-id guard baked in. Refactor CatalogView to consume it (category buttons can push a URL into a ref now - the composable follows). Watch forty lines leave the component.
2. Build `SearchView.vue` at `/search`: an input (`v-model.trim`) + `useFetch` with a getter URL against `https://dummyjson.com/products/search?q=...&limit=8`, and all four lanes (typing gibberish exercises "empty" beautifully). Nav link, obviously.
3. Feel the missing debounce: Network tab open, type "chocolate" naturally, count the requests. Add the deep dive's `useDebouncedRef` (or a `setTimeout` inline if you prefer to hand-roll once) and count again.
4. Prove instance independence: mount two search boxes on one page (import SearchView's guts into a quick second component, or just add a second input+useFetch pair). Different queries, different results, zero crosstalk - *composables instantiate*. Then say out loud where the results would live if you wanted the two boxes to *share* one result set. (Module 6 echoes.)
5. Read one VueUse source file - `useToggle` is 20 lines - and confirm it's nothing but this lesson's conventions.

<details>
<summary>💡 Hint - the reactive useFetch, assembled</summary>

```js
import { ref, watch, toValue } from 'vue'

export function useFetch(urlSource) {
  const data = ref(null), error = ref(null), loading = ref(false)
  let requestId = 0

  async function load() {
    const id = ++requestId
    loading.value = true
    error.value = null
    try {
      const res = await fetch(toValue(urlSource))
      if (!res.ok) throw new Error(`HTTP ${res.status}`)
      const json = await res.json()
      if (id === requestId) data.value = json
    } catch (e) {
      if (id === requestId) error.value = e.message
    } finally {
      if (id === requestId) loading.value = false
    }
  }

  watch(() => toValue(urlSource), load, { immediate: true })
  return { data, error, loading, retry: load }
}
```

</details>

<details>
<summary>✅ Solution - SearchView.vue</summary>

```vue
<script setup>
import { ref } from 'vue'
import { useFetch } from '../composables/useFetch'
import { useDebouncedRef } from '../composables/useDebouncedRef'

const q = ref('')
const slowQ = useDebouncedRef(q, 300)

const { data, error, loading } = useFetch(
  () => `https://dummyjson.com/products/search?q=${encodeURIComponent(slowQ.value)}&limit=8`
)
</script>

<template>
  <h1>Search</h1>
  <input v-model.trim="q" placeholder="Search products...">

  <p v-if="loading">Searching... ⏳</p>
  <p v-else-if="error" class="error">Search failed: {{ error }}</p>
  <p v-else-if="data && data.products.length === 0">No matches for "{{ q }}" 🤷</p>
  <ul v-else-if="data">
    <li v-for="p in data.products" :key="p.id">{{ p.title }} - ${{ p.price }}</li>
  </ul>
</template>
```

Step 4's answer: a store - shared result state across components is Module 6's flowchart saying "store" (and "a products cache store whose actions call useFetch-style logic" is exactly how larger apps structure it).

</details>

## ✋ Checkpoint

1. Component, composable, or store? (a) a card with a title slot, (b) "fetch + three states" logic, (c) the cart, (d) tracking the mouse position, (e) the current user.
2. Two components each call `useFetch(differentUrls)`. Then two components each call `useCartStore()`. Contrast what they get.
3. Why does the reactive useFetch take `() => url.value` (a getter) rather than `url.value` directly - which module's lesson is this again, in disguise?
4. Name the two rough edges of search-as-you-type and the fix for each.

<details>
<summary>Answers</summary>

1. (a) component - it's UI. (b) composable - stateful logic, fresh per use. (c) store - shared. (d) composable (VueUse's useMouse, in fact). (e) store - shared identity.
2. useFetch: two independent instances, separate refs - composables instantiate. useCartStore: the same singleton twice - stores share.
3. Passing `url.value` hands over a dead string snapshot; the getter lets the composable *re-read* reactively - it's Module 2's "destructured/evaluated values go dead" and Module 7 lesson 2's watch-a-getter, unified.
4. The stale-response race (fix: abort or id-guard, buried in the composable) and request volume per keystroke (fix: debounce).

</details>

## 📚 Further reading

- [Composables - Vue docs](https://vuejs.org/guide/reusability/composables.html) - the official guide; its example is literally useFetch
- [VueUse](https://vueuse.org) - browse for ten minutes; you'll recognise the shape of everything now

---

⬅️ [Previous: Loading, error, success](./02-loading-error-success.md) · 🏠 [Course home](../README.md) · ➡️ Next: [Module 8 · Capstone](../module-08-capstone/)
