# 1 · Fetching data into a component

> **You'll learn:** how to pull real data from an API into a component's refs - the moment your app stops being an island.

## Why this matters

Every app so far ran on arrays you typed by hand. Real apps get their data from servers: products from a database, users from an auth service, weather from someone else's computer entirely. The bridge is HTTP + JSON + `fetch`, and the Vue side is delightfully small - the data lands in refs, and everything you've built all course takes over from there.

## The big picture

```vue
<script setup>
import { ref, onMounted } from 'vue'

const users = ref([])

onMounted(async () => {
  const response = await fetch('https://jsonplaceholder.typicode.com/users')
  users.value = await response.json()
})
</script>

<template>
  <ul>
    <li v-for="u in users" :key="u.id">{{ u.name }} - {{ u.email }}</li>
  </ul>
</template>
```

Run it: a list of (fake but properly served) users appears a beat after the page. The anatomy: an empty ref, a fetch when the component appears, and assignment - **once data is in a ref, the API part is over** and Module 1's `v-for` renders it like any other array.

(JSONPlaceholder is a free practice API - real HTTP, canned data, no key needed. This module also uses [dummyjson.com](https://dummyjson.com), which has products and search.)

## The moving parts, one at a time

**`fetch(url)`** - the browser's built-in HTTP client. It returns a *promise* (the data takes real milliseconds to cross the internet; JavaScript doesn't wait around), and **`await`** is how you write "pause here until it arrives" - legal inside any `async` function.

**`.json()`** - the response arrives as raw bytes; this parses them into JavaScript objects (also async - hence the second await).

**`onMounted(callback)`** - a Vue *lifecycle hook*: run this when the component has appeared on the page. It's the natural "go get the data" moment, and it's new vocabulary worth a beat: components are born (mounted), live (update), and die (unmounted - remember route changes in Module 5?), and you can hook code to each. Fetching is 90% of what you'll use hooks for at this stage.

```js
import { onMounted, onUnmounted } from 'vue'
onMounted(() => console.log('component on stage'))
onUnmounted(() => console.log('and off again'))     // cleanup lives here, when you need it
```

> [!NOTE]
> Why not just call the fetch at the top of `<script setup>`? An `await` at the *top level* of setup turns the whole component async (needing `<Suspense>` machinery you don't want yet) - and un-awaited floating promises make errors untraceable. `onMounted(async () => ...)` keeps setup synchronous and the fetch supervised. It's the course's default; adjust when you meet Suspense someday.

## Query strings: asking for exactly what you want

APIs parameterize through the URL - the same query strings Module 5 read, now written:

```js
const response = await fetch('https://dummyjson.com/products/search?q=phone&limit=5')
const data = await response.json()
products.value = data.products          // this API nests the array one level down
```

Two habits that keep this professional: read each API's docs for its *response shape* (is the array at the top, or under `.products`? nobody guesses right), and build URLs safely once terms come from users:

```js
const url = `https://dummyjson.com/products/search?q=${encodeURIComponent(term)}&limit=5`
```

`encodeURIComponent` armours spaces, `&`s, and emoji so a search for "salt & pepper 🧂" doesn't shred your query string.

<details>
<summary>🔍 Deep dive: what fetch actually kicks off</summary>

One `fetch` call: the browser resolves the domain to an address (DNS), opens a TLS-encrypted connection, sends a `GET /users HTTP/2` request with headers, and the server - typically after querying a real database - streams back a status line (`200 OK`), headers, and the JSON body. Milliseconds, dozens of steps, and *any* of them can fail: airplane mode kills DNS, the server 500s, the JSON arrives truncated. That's not trivia - it's the syllabus for lesson 2, because code that assumes the happy path is code that lies to users. Open your browser devtools' **Network tab** while the exercise runs and watch the whole drama: status, timing waterfall, response body. That tab is where API debugging lives.

Also worth naming: `fetch` defaults to GET (reading). Sending data - `POST` with a JSON body - is the same function with an options object (`method`, `headers`, `body`); dummyjson accepts practice POSTs when you're curious. The capstone sticks to reading, which is most of most apps anyway.

</details>

## 🛠️ Try it - real products for the Snack Shop

Time to fire the hand-typed array (gently - it gets a pension):

1. New view + route: `CatalogView.vue` at `/catalog`, nav link included (Module 5 muscle memory). In it: fetch `https://dummyjson.com/products?limit=12` on mount and render title, price, and thumbnail (`<img :src="p.thumbnail">` - `:` forever) in a grid of UiCards.
2. Watch it actually happen: Network tab open, refresh. Find the request, click it, read the JSON. Then throttle ("Slow 4G" in the Network tab's dropdown) and refresh - *see* the blank moment your users on trains see. (That discomfort is lesson 2's brief.)
3. Add a category filter the API way: buttons for two or three categories fetching `https://dummyjson.com/products/category/beauty` etc. on click - a fetch triggered by an *event* rather than mount; same pattern, different trigger. Notice the old results linger until new ones land... noted, lesson 2.
4. Sharpen the shape-reading habit: this API returns `{ products: [...], total: 194, ... }` - render "Showing 12 of 194" from the envelope. The docs-then-destructure minute at the start of any API work repays itself forever.

<details>
<summary>💡 Hint - the fetch-on-click variant</summary>

```js
const products = ref([])
const total = ref(0)

async function load(url) {
  const res = await fetch(url)
  const data = await res.json()
  products.value = data.products
  total.value = data.total
}

onMounted(() => load('https://dummyjson.com/products?limit=12'))
// buttons: @click="load('https://dummyjson.com/products/category/beauty')"
```

One loader, two triggers - factoring you'll be very glad of in lesson 3.

</details>

<details>
<summary>✅ Solution - CatalogView.vue</summary>

```vue
<script setup>
import { ref, onMounted } from 'vue'
import UiCard from '../components/UiCard.vue'

const products = ref([])
const total = ref(0)

async function load(url) {
  const res = await fetch(url)
  const data = await res.json()
  products.value = data.products
  total.value = data.total
}

onMounted(() => load('https://dummyjson.com/products?limit=12'))
</script>

<template>
  <h1>Catalog</h1>
  <p>Showing {{ products.length }} of {{ total }}</p>
  <button @click="load('https://dummyjson.com/products/category/beauty')">Beauty</button>
  <button @click="load('https://dummyjson.com/products/category/groceries')">Groceries</button>
  <button @click="load('https://dummyjson.com/products?limit=12')">All</button>

  <div class="grid">
    <UiCard v-for="p in products" :key="p.id">
      <template #title>{{ p.title }}</template>
      <img :src="p.thumbnail" :alt="p.title" width="120">
      <p>${{ p.price }}</p>
    </UiCard>
  </div>
</template>

<style scoped>
.grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(180px, 1fr)); gap: 0.5rem; }
</style>
```

</details>

## ✋ Checkpoint

1. Order these five into the working sequence: assign to ref, await response.json(), component mounts, await fetch(url), template re-renders.
2. Why does the fetch live in `onMounted` rather than bare in `<script setup>` - two reasons from the lesson.
3. `console.log(products.value)` right *after* the `await response.json()` line shows the goods, but a log placed right after `onMounted(...)` in setup shows `[]`. Both correct - explain.
4. A user searches for `100% cotton & silk`. What breaks in `` `...?q=${term}` `` and what's the fix?

<details>
<summary>Answers</summary>

1. Component mounts → await fetch(url) → await response.json() → assign to ref → template re-renders.
2. A top-level await would make the component async (Suspense territory), and onMounted keeps the promise supervised at a well-defined lifecycle moment.
3. The setup-level log runs *before* mounting (it only registered the hook); the fetch hasn't happened yet. Async means "later" - logs are timestamps, not positions.
4. The `&` splits the query string (`silk` becomes a separate parameter) and `%` can corrupt decoding. `encodeURIComponent(term)` armours it.

</details>

## 📚 Further reading

- [Using the Fetch API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) - the canonical reference, including POST bodies
- [Lifecycle Hooks - Vue docs](https://vuejs.org/guide/essentials/lifecycle.html) - the full birth-to-unmount diagram

---

⬅️ [Module home](./README.md) · 🏠 [Course home](../README.md) · ➡️ [Next: Loading, error, success](./02-loading-error-success.md)
