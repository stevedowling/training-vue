# 2 · Loading, error, success - handling all three honestly

> **You'll learn:** the three-state pattern every data-driven page needs - so slow networks show spinners, failures show apologies, and only actual success shows data.

## Why this matters

Lesson 1's code works beautifully on your machine and lies everywhere else: on a train, the user stares at a blank grid wondering if the app is broken; when the API hiccups, they get silence and a console error they'll never see. Every fetch has three possible presents - *still waiting, failed, succeeded* - and honest UIs render all three. This pattern is so universal that once you see it, you'll notice every app you use doing it (or painfully not).

## The big picture

Three refs tell the truth; the template has a lane for each:

```vue
<script setup>
import { ref, onMounted } from 'vue'

const data = ref(null)
const loading = ref(false)
const error = ref(null)

onMounted(async () => {
  loading.value = true
  error.value = null
  try {
    const res = await fetch('https://dummyjson.com/products?limit=12')
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    data.value = await res.json()
  } catch (e) {
    error.value = e.message
  } finally {
    loading.value = false
  }
})
</script>

<template>
  <p v-if="loading">Loading... ⏳</p>
  <p v-else-if="error" class="error">Couldn't load products: {{ error }} 😞</p>
  <div v-else-if="data">
    <!-- the happy grid from lesson 1 -->
  </div>
</template>
```

Read the `v-if` chain as a protocol: **loading wins, then error, then data**. Exactly one lane renders, the states can't contradict each other on screen, and the blank-mystery moment is gone.

## The two failure families (and the trap between them)

Here's the part everyone learns the hard way, so learn it the easy way: **`fetch` does not throw on HTTP errors.** A 404 or a 500 is, to fetch, a successful conversation - the server *answered*, the answer just happens to be "no". Only *network-level* failure (offline, DNS dead, server unreachable) rejects the promise:

| Failure | What fetch does | Caught by |
|---|---|---|
| No internet, server down | promise rejects | `catch` |
| 404, 500, 403... | resolves fine, `res.ok === false` | **you**, checking `res.ok` |

Hence the line that makes the pattern honest:

```js
if (!res.ok) throw new Error(`HTTP ${res.status}`)
```

Throwing it yourself funnels both families into one `catch`, one `error` ref, one apology lane. Forget this line and your 404s render as *eternal loading* or a cryptic JSON-parse crash - the two great fetch mysteries, both solved by four words.

`finally` completes the trio: loading must end *whichever* way the story went, and `finally` runs after both success and catch - one un-forgettable place to put `loading.value = false`.

## Craft: the states beyond the states

Three touches separate adequate from good:

- **The empty success.** `data` arrived and it's... `[]`. That's not an error and not loading - it's a fourth lane worth a friendly line: `<p v-else-if="data.products.length === 0">Nothing matched 🤷</p>` before the grid. (Searches hit this constantly - next lesson cares.)
- **Errors offer a way forward.** A retry button costs one line - the fetch is already a function:

```vue
<p v-else-if="error">Couldn't load. <button @click="load">Try again</button></p>
```

- **Stale data during refetch.** Lesson 1's category buttons left old products up while new ones loaded. Decide *deliberately*: clear the grid (simple), or keep it dimmed under a small spinner (`v-if="loading && !data"` for the big spinner, an inline one otherwise - the "stale-while-revalidate" feel modern apps favour). Either is fine; *un*decided is what looks broken.

> [!TIP]
> Test your three lanes on demand: the error lane via a garbage URL (`https://dummyjson.com/nope` → 404 exercises your `res.ok` line) or devtools' Network tab set to "Offline" (exercises the catch). The loading lane via "Slow 4G" throttling. Thirty seconds of sabotage per page - make it a habit before every "done".

<details>
<summary>🔍 Deep dive: the race condition hiding in refetch</summary>

Click "Beauty", then quickly "Groceries": two fetches are airborne. If Beauty's response arrives *last* (networks reorder freely), it overwrites Groceries - the UI shows the wrong catalogue under the right button. This is the **stale response race**, and every senior dev has been bitten.

The modern fix is `AbortController` - cancel the previous request when starting a new one:

```js
let controller = null

async function load(url) {
  controller?.abort()                        // kill any in-flight predecessor
  controller = new AbortController()
  try {
    const res = await fetch(url, { signal: controller.signal })
    // ...
  } catch (e) {
    if (e.name === 'AbortError') return      // our own abort - not an error, just superseded
    error.value = e.message
  }
}
```

(A simpler guard - track a request id and ignore stale arrivals - also works and needs no new API.) You don't have to implement this today; you *do* have to recognise the bug's shape, because lesson 3's search box mass-produces exactly this race, and the composable is a tidy place to bury the fix once.

</details>

## 🛠️ Try it - harden the catalog

Take lesson 1's CatalogView from demo to trustworthy:

1. Retrofit the full pattern: `loading` / `error` / data refs, `res.ok` check, try/catch/finally, and the four template lanes (loading, error + retry button, empty, grid).
2. Sabotage tour, one lane at a time: throttle to Slow 4G (watch loading), point one category button at `/products/category/does-not-exist` (this API returns an empty list - meet your empty lane), change the domain to something dead with Network "Offline" (watch catch + retry work). Fix the button back afterwards.
3. Decide your stale-data policy for the category buttons and implement it - clear-on-load or keep-while-loading. One sentence in a comment saying which and why.
4. Stretch: reproduce the deep dive's race on purpose - throttle hard, click two categories fast, watch the wrong winner. Then fix it (request-id guard is fine). Nothing builds respect for a bug like meeting it.

<details>
<summary>💡 Hint - the request-id guard (simpler than AbortController)</summary>

```js
let requestId = 0

async function load(url) {
  const id = ++requestId
  loading.value = true
  try {
    const res = await fetch(url)
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    const json = await res.json()
    if (id !== requestId) return        // a newer request superseded us - discard
    data.value = json
  } catch (e) {
    if (id === requestId) error.value = e.message
  } finally {
    if (id === requestId) loading.value = false
  }
}
```

</details>

<details>
<summary>✅ Solution - the hardened template</summary>

```vue
<template>
  <h1>Catalog</h1>
  <button @click="load(ALL_URL)">All</button>
  <button @click="load(CAT_URL('beauty'))">Beauty</button>
  <button @click="load(CAT_URL('groceries'))">Groceries</button>

  <p v-if="loading">Loading... ⏳</p>
  <p v-else-if="error" class="error">
    Couldn't load products ({{ error }}).
    <button @click="load(lastUrl)">Try again</button>
  </p>
  <p v-else-if="products.length === 0">Nothing in this aisle 🤷</p>
  <div v-else class="grid"><!-- lesson 1's grid --></div>
</template>
```

With `lastUrl` remembered inside `load()` so retry repeats the *failed* request, not the default one - the small touches are the lesson. Script: the big-picture skeleton plus the hint's id guard if you did step 4.

</details>

## ✋ Checkpoint

1. The API returns 500. Walk the big-picture code: which lines run, what does each of the three refs hold at the end, and what does the user see?
2. Same question, airplane mode.
3. A page shows "Loading..." forever after a failed request. Which *two* different bugs from this lesson produce that symptom?
4. Why does `loading.value = false` live in `finally` instead of appearing twice (after success, and in catch)?

<details>
<summary>Answers</summary>

1. fetch resolves (the server answered!), `res.ok` is false, your throw fires, catch sets `error = "HTTP 500"`, finally clears loading. Refs: data null, loading false, error set. User sees the apology + retry.
2. fetch itself rejects, catch sets error to the network message, finally clears loading - same end state, different door in. (That convergence is the design.)
3. Missing `res.ok` check (a 404's JSON-shaped "no" never populates data, error stays null... though with the full pattern loading still clears - the *classic* eternal-loading is bug two:) missing `finally`/forgotten `loading = false` on the error path.
4. Because it must run on *every* exit path, including ones added later - finally is the only spot that can't be forgotten by a future editor adding an early return.

</details>

## 📚 Further reading

- [Response.ok - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Response/ok) - four words, hours of debugging saved
- [AbortController - MDN](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) - the deep dive's cancellation, properly documented

---

⬅️ [Previous: Fetching data](./01-fetching-data.md) · 🏠 [Course home](../README.md) · ➡️ [Next: Composables](./03-composables.md)
