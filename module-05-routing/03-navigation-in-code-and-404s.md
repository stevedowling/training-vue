# 3 · Navigation in code + 404s

> **You'll learn:** how to navigate from JavaScript (redirect after saving, cancel back to the list) and how to catch every URL you don't recognise with a proper 404 page.

## Why this matters

Links cover navigation the *user* initiates - but plenty of navigation is the *app's* decision: land on the order page after checkout, bounce to the list after deleting, return to login when a session dies. And the flip side of owning your URLs is owning the bad ones: an app with real links will be visited at broken ones, and "blank white page" is not an answer. These two close out routing.

## The big picture

```vue
<script setup>
import { useRouter } from 'vue-router'

const router = useRouter()

function placeOrder() {
  // ...validate, save (Module 4 did this part)...
  router.push('/thanks')          // navigation as a statement, not a click
}
</script>
```

`router.push(path)` does exactly what clicking a RouterLink to that path does - history entry, view swap, no reload. Note the sibling: **`useRoute()`** (no r) is *where am I* - read-only info, last lesson's tool. **`useRouter()`** is *take me somewhere* - the actions object. You'll typo one for the other exactly once per project.

## The navigation toolkit

```js
router.push('/shop')                        // go (adds a history entry)
router.push(`/products/${saved.id}`)        // computed destinations, obviously
router.replace('/login')                    // go WITHOUT adding history
router.back()                               // programmatic back button
```

`push` vs `replace` is about the back button's story. After `push('/thanks')`, back returns to the checkout form - fine. After a login redirect done with `push`, back returns to... the login page you just left, trapping the user in a loop; `replace` swaps the current entry instead, so back skips the intermediate step. The question to ask: *"would going back to the page I'm leaving make sense?"* No → replace.

```js
// the classic pairing, in a checkout handler
function placeOrder() {
  if (!formValid.value) return
  orders.push(buildOrder())
  router.push(`/orders/${newId}`)    // push: back-to-the-form is a reasonable wish
}

// the classic replace, after a redirect-because-not-allowed
router.replace('/login')             // back should NOT revisit the forbidden page
```

## Catching everything else: the 404 route

One special path pattern matches any URL nothing else claimed:

```js
// router/index.js - LAST in the routes array by convention
{ path: '/:pathMatch(.*)*', component: NotFoundView }
```

(The incantation reads: a param named `pathMatch`, matching anything including slashes. Nobody memorizes it; everybody pastes it - now you know what it says.)

`NotFoundView.vue` is just a page - and a good one is a small act of hospitality:

```vue
<template>
  <h1>404 - nothing here 🕳️</h1>
  <p>The address <code>{{ $route.fullPath }}</code> doesn't match anything we know.</p>
  <RouterLink to="/">← Back to safety</RouterLink>
</template>
```

Related and better than v-else limbo: lesson 2's ProductView showed "No such product" for `/products/999` - a *valid route* with an *invalid id*. The polished move is redirecting such cases to the 404:

```js
watch(product, (p) => {
  if (!p) router.replace('/not-found')     // replace: don't leave the dead URL in history
}, { immediate: true })
```

(Which needs `/not-found` as a normal route to the same NotFoundView - catch-all for unknown *paths*, explicit route for known-path-unknown-*thing*.)

> [!TIP]
> Watch + immediate + replace: three modules' tools in one three-line idiom. This is the course working as intended - new concepts per lesson shrink because old ones compose.

<details>
<summary>🔍 Deep dive: navigation guards - the router's bouncers</summary>

The third navigation trigger (after links and pushes) is *interception*: code that runs **before** a navigation lands and may veto or redirect it. Route-level:

```js
{
  path: '/admin',
  component: AdminView,
  beforeEnter: (to) => {
    if (!isLoggedIn.value) return '/login'    // redirect instead
  }
}
```

Or global - `router.beforeEach((to) => ...)` runs for *every* navigation, the standard home of auth checks ("does this route's `meta.requiresAuth` demand a user?"). Guards can also ask "unsaved changes, really leave?" via `onBeforeRouteLeave` inside a component. You don't need guards until the app has accounts or drafts - but when the capstone or a real project grows either, this is the search term: **navigation guards**. The mental model is a bouncer at the door of every route change; the docs page is excellent.

</details>

## 🛠️ Try it - close the loops

Three small jobs finish the routing story:

1. **The 404**: add `NotFoundView.vue` (make it charming - it's the page nobody wants, so it may as well have personality), the catch-all route, and a `/not-found` alias route. Visit `/sandwich/of/lies` to test.
2. **Redirect the dead ids**: upgrade ProductView with the watch-and-replace idiom so `/products/999` lands on the 404 instead of the shrug paragraph. Confirm the back button *doesn't* return you to the dead URL (that's `replace` earning its place).
3. **Programmatic checkout**: give ShopView a "Checkout" FancyButton, enabled only when `itemCount > 0`, that pushes to a new `/thanks` route. `ThanksView.vue` shows a warm confirmation and - the catch - the order summary. Except... the cart's refs live in ShopView, which just unmounted. Pass the essentials as *query* (`router.push({ path: '/thanks', query: { items: itemCount.value, total: total.value } })`) and read them from `route.query` on the other side. Feel free to grumble - shipping state through URLs is exactly the clunk Module 6 exists to remove, and the grumble is the curriculum.
4. Victory lap: click through every page, deep-link a product, mistype a URL, checkout, hit back everywhere. The sandbox is a *site* now.

<details>
<summary>💡 Hint - push with query, and reading it back</summary>

`router.push()` accepts an object: `{ path: '/thanks', query: { items: 3, total: 16 } }` becomes `/thanks?items=3&total=16`. ThanksView reads `route.query.items` - strings again, of course, so `Number(...)` before math (that's the third time this module; it's a habit now).

</details>

<details>
<summary>✅ Solution - the new routes and the redirect</summary>

```js
// router/index.js additions
import NotFoundView from '../views/NotFoundView.vue'
import ThanksView from '../views/ThanksView.vue'

routes: [
  // ...existing...
  { path: '/thanks', component: ThanksView },
  { path: '/not-found', component: NotFoundView },
  { path: '/:pathMatch(.*)*', component: NotFoundView },   // keep last
]
```

```js
// ProductView.vue - the dead-id redirect
import { useRoute, useRouter } from 'vue-router'
const route = useRoute()
const router = useRouter()
const product = computed(() => products.find(p => p.id === Number(route.params.id)))

watch(product, (p) => { if (!p) router.replace('/not-found') }, { immediate: true })
```

```js
// ShopView.vue - checkout
function checkout() {
  router.push({ path: '/thanks', query: { items: itemCount.value, total: total.value } })
}
```

ThanksView template: `<p>{{ route.query.items }} items - ${{ route.query.total }}. On their way! 🚀</p>` plus a RouterLink home.

</details>

## ✋ Checkpoint

1. `useRoute` vs `useRouter` - one job description each, and which one lesson 2 used.
2. Push or replace, with the back-button reasoning: (a) after successful checkout → receipt page, (b) `/products/999` → 404, (c) clicking a product in the list.
3. Why must the catch-all route sit last in the routes array... or must it? (Think about how specific `/:pathMatch(.*)*` is versus `/shop`.)
4. The capstone will ask this, so rehearse: name the three ways a navigation can start in a finished Vue app.

<details>
<summary>Answers</summary>

1. `useRoute()`: where am I - params, query, path (lesson 2's tool). `useRouter()`: take me somewhere - push, replace, back.
2. (a) push - backing into the form is legitimate. (b) replace - the dead URL shouldn't survive in history. (c) push (via RouterLink) - back-to-the-list is the most-pressed button on the internet.
3. Trick question, mostly: Vue Router ranks routes by specificity, not order, so `/shop` beats the catch-all wherever it sits - but convention keeps the catch-all last because humans read the array top-down and "everything else → 404" belongs at the bottom of that story.
4. A RouterLink click, a programmatic push/replace/back, and the browser's own history buttons - all three land in the same router, which is why behaviour stays consistent.

</details>

## 📚 Further reading

- [Programmatic Navigation - Vue Router docs](https://router.vuejs.org/guide/essentials/navigation.html) - push/replace/go and their object forms
- [Navigation Guards - Vue Router docs](https://router.vuejs.org/guide/advanced/navigation-guards.html) - the deep dive's bouncers, for the day auth arrives

---

⬅️ [Previous: Dynamic routes](./02-dynamic-routes.md) · 🏠 [Course home](../README.md) · ➡️ Next: [Module 6 · App-wide State](../module-06-app-wide-state/)
