# 1 · Your First Component - extracting and reusing

> **You'll learn:** how to split a growing page into components of your own, and develop a feel for *where* to cut.

## Why this matters

Every app you've built so far lives in one file, and `App.vue` is starting to feel crowded - real apps have hundreds of times more UI. Components are how Vue apps scale: small, self-contained pieces with their own logic, markup, and styles, composed like Lego. You've been *using* components since Module 1 (`App.vue` is one); now you'll make your own.

## The big picture

A component is a `.vue` file. Using it is three steps: create, import, place:

```vue
<!-- src/components/AppFooter.vue -->
<script setup>
import { ref } from 'vue'
const year = ref(new Date().getFullYear())
</script>

<template>
  <footer>Built with Vue, {{ year }} 🌱</footer>
</template>
```

```vue
<!-- src/App.vue -->
<script setup>
import AppFooter from './components/AppFooter.vue'
</script>

<template>
  <h1>My App</h1>
  <AppFooter />       <!-- your own tag! -->
</template>
```

`<AppFooter />` now works exactly like `<footer>` or `<button>` - except *you* define what it is. That's the whole mechanism; the rest of the module is about making these tags talk to each other.

## The rules of the road

- **Files live in `src/components/`**, one component per file (the scaffold made the folder for you; nested folders are fine as apps grow).
- **Name with two or more words**, `PascalCase`: `AppFooter`, `ProductRow`, `QuestButton`. Single words like `<Header>` risk colliding with current or future *HTML* tags - multi-word names never can.
- **Each component is an island.** Its refs, computeds, and functions are its own - two copies of a component on one page each have their *own* state. (You'll prove this in the exercise, and it's a feature, not a surprise.)

## Where to cut: the three signs

Splitting too little leaves you with a 500-line `App.vue`; splitting too much drowns you in files. Cut when you see any of these:

| Sign | Example |
|---|---|
| **Repetition** - the same chunk of template appears twice | two product rows in Snack Cart |
| **A nameable thing** - you'd naturally call it something | "that's the search bar", "that's a result card" |
| **Independent reasoning** - you could describe what it does without mentioning the rest of the page | a footer, a nav menu, a star rating |

> [!TIP]
> When in doubt, wait. Extracting a component later is a five-minute copy-paste job - you don't have to guess right up front. The repetition sign in particular can be trusted: the *second* time you paste the same template chunk, that's the component announcing itself.

## Scoped styles: CSS that stays home

A component's `<style>` block leaks to the whole page by default - add `scoped` and its rules apply only to this component's own template:

```vue
<style scoped>
footer {
  color: #42b883;   /* styles THIS footer, not every footer on the page */
}
</style>
```

This kills the classic CSS problem (a `.card` rule in one corner breaking cards everywhere) at the language level. Habit worth forming now: `scoped` on every component style block unless you have a stated reason otherwise.

<details>
<summary>🔍 Deep dive: what a component actually is at runtime</summary>

When Vite processes `AppFooter.vue`, the SFC becomes a plain JavaScript object: the compiled template as a render function, plus its setup logic. `<AppFooter />` in a parent template tells Vue "create an *instance* of that object here" - running its `<script setup>` fresh, with its own refs. That's precisely why two `<AppFooter />` tags don't share state: two instances, two runs of setup, two sets of refs. It's also why Module 2's reactivity just works per-component: each instance's render function is one of those tracked "effects", re-running when its own dependencies change - and *only* its own.

</details>

## 🛠️ Try it - break up the sandbox

Take your `vue-sandbox` (whatever `App.vue` currently holds - Click Quest, Snack Cart, or both) and give it structure:

1. Create `AppHeader.vue`: an `<h1>` with your app's name plus a tagline `<p>`, styled with a `scoped` block. Use it at the top of `App.vue`.
2. Create `AppFooter.vue` as in the lesson. Use it at the bottom.
3. The island test: create `ClickCounter.vue` - a self-contained mini counter (one ref, one button, from Module 1 memory). Place it in `App.vue` **twice**. Click one - does the other move? Write down why in a comment.
4. The cliffhanger: try to extract one Snack Cart product row into `ProductRow.vue`. You'll hit a wall almost immediately - the row needs to know *which* product it is, and the parent needs to know when *its* buttons are clicked. Get it rendering something static, feel the limitation, and leave a `<!-- TODO: needs props (next lesson!) -->`.

<details>
<summary>💡 Hint - the counter island</summary>

```vue
<script setup>
import { ref } from 'vue'
const count = ref(0)
</script>

<template>
  <button @click="count++">Clicked {{ count }} times</button>
</template>
```

Import once, place twice. Independent instances, independent counts.

</details>

<details>
<summary>✅ What App.vue should look like when you're done</summary>

```vue
<script setup>
import AppHeader from './components/AppHeader.vue'
import AppFooter from './components/AppFooter.vue'
import ClickCounter from './components/ClickCounter.vue'
</script>

<template>
  <AppHeader />

  <ClickCounter />
  <ClickCounter />   <!-- separate instance, separate count: components are islands -->

  <!-- your existing Snack Cart / Click Quest content -->

  <AppFooter />
</template>
```

</details>

## ✋ Checkpoint

1. Two `<ClickCounter />` tags on one page. One shows 7, the other 2. Bug or feature - and what explains it?
2. Why does Vue's style guide insist on `ProductRow` over `Product` as a component name?
3. A component's `<style>` block sets `p { margin: 0 }` and suddenly paragraphs across the whole app lost their margins. What one word was missing?
4. You're staring at a 300-line `App.vue` wondering where to cut. Name the three signs from this lesson.

<details>
<summary>Answers</summary>

1. Feature - each tag creates its own instance with its own refs. Shared state across components is exactly what props (next lesson) and stores (Module 6) are for.
2. Multi-word names can never collide with HTML's own tags, present or future - `<product>` might mean something to browsers someday; `<product-row>` never will.
3. `scoped` - without it, a component's styles apply page-wide.
4. Repetition, a nameable thing, independent reasoning - any one is enough to justify the cut.

</details>

## 📚 Further reading

- [Components Basics - Vue docs](https://vuejs.org/guide/essentials/component-basics.html) - this lesson and the next two, in official form
- [Style Guide - Vue docs](https://vuejs.org/style-guide/) - the naming conventions, with reasoning

---

⬅️ [Module home](./README.md) · 🏠 [Course home](../README.md) · ➡️ [Next: Props](./02-props.md)
