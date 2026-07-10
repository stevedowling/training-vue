# 3 · Templates & Reactivity

> **You'll learn:** the four template tools you'll use in every Vue component ever - showing data, reacting to events, conditional display, and lists - and build your first real page with them.

## Why this matters

Almost everything a UI does boils down to: *show some data, respond when the user does something, show/hide things, repeat things*. Vue has one template feature for each. Learn these four and you can read most Vue components in the wild.

## The big picture

| You want to... | Template syntax | Example |
|---|---|---|
| Show data | `{{ }}` | `<p>{{ count }}</p>` |
| React to an event | `@click` | `<button @click="count++">` |
| Show/hide | `v-if` / `v-else` | `<p v-if="count > 10">Wow</p>` |
| Repeat for a list | `v-for` | `<li v-for="item in items">` |

And one script-side idea powering it all: wrapping data in `ref()` is what makes Vue *watch* it.

## `ref()` - making data reactive

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)        // reactive - Vue watches it
const appName = 'Sandbox'   // plain constant - fine if it never changes
</script>
```

`ref(0)` wraps the value in a reactive container. Two rules:

- **In `<script>`**: read/write via `.value` → `count.value++`
- **In `<template>`**: no `.value` needed → `{{ count }}` (Vue unwraps it for you)

> [!WARNING]
> Forgetting `.value` in script code is *the* classic Vue beginner bug. If a value "just won't update", check for a missing `.value` first.

## Events with `@`

```vue
<template>
  <!-- inline, for one-liners -->
  <button @click="count++">+1</button>

  <!-- or call a function, for anything bigger -->
  <button @click="reset">Reset</button>
</template>

<script setup>
import { ref } from 'vue'
const count = ref(0)

function reset() {
  count.value = 0   // note: .value, we're in script land
}
</script>
```

`@click` is shorthand for `v-on:click`; there's also `@input`, `@submit`, `@keyup` - any DOM event.

## Showing and hiding with `v-if`

```vue
<template>
  <p v-if="count === 0">Click the button to start!</p>
  <p v-else-if="count < 10">Keep going... ({{ count }})</p>
  <p v-else>You made it! 🎉</p>
</template>
```

Exactly one of these renders at a time, and it switches automatically as `count` changes - reactivity again, no extra wiring.

## Lists with `v-for`

```vue
<script setup>
import { ref } from 'vue'
const todos = ref(['Learn templates', 'Do the exercise', 'Tick the checkbox'])
</script>

<template>
  <ul>
    <li v-for="(todo, index) in todos" :key="index">{{ todo }}</li>
  </ul>
  <button @click="todos.push('Another one')">Add</button>
</template>
```

Push to the array, the list grows on screen. The `:key` attribute gives each item a stable identity so Vue can update the list efficiently - you'll get a console warning without it.

<details>
<summary>🔍 Deep dive: what's the colon in <code>:key</code>?</summary>

`:key="index"` is shorthand for `v-bind:key="index"` - it makes an attribute *dynamic* (a JavaScript expression) instead of a literal string. Compare `<img src="photo.jpg">` (always that file) with `<img :src="currentPhoto">` (whatever the variable holds). You'll use `:` constantly from Module 3 onward; for now, just recognise it.

</details>

## 🛠️ Try it - build "Click Quest"

In your `vue-sandbox` project, replace `App.vue` with a tiny clicking game, built from everything above:

- A heading and a **click counter button** showing its own count
- A **milestone message** that changes: 0 clicks → "Start clicking!", 1–9 → "Warming up...", 10+ → "Legendary. 🏆" (`v-if` / `v-else-if` / `v-else`)
- An **achievement log**: every time the count hits a multiple of 5, push `"Reached 5!"` etc. onto a list, rendered with `v-for`
- A **reset button**

<details>
<summary>💡 Hint 1 - the shape of the script</summary>

You need two refs: `count` (number) and `achievements` (array). Handle the click in a function - it needs to do two things (increment, and maybe push an achievement), which is too much for an inline handler.

</details>

<details>
<summary>💡 Hint 2 - detecting multiples of 5</summary>

After incrementing: `if (count.value % 5 === 0) { achievements.value.push(`Reached ${count.value}!`) }`

</details>

<details>
<summary>✅ Solution</summary>

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)
const achievements = ref([])

function handleClick() {
  count.value++
  if (count.value % 5 === 0) {
    achievements.value.push(`Reached ${count.value}!`)
  }
}

function reset() {
  count.value = 0
  achievements.value = []
}
</script>

<template>
  <h1>Click Quest</h1>

  <button @click="handleClick">Clicks: {{ count }}</button>
  <button @click="reset">Reset</button>

  <p v-if="count === 0">Start clicking!</p>
  <p v-else-if="count < 10">Warming up...</p>
  <p v-else>Legendary. 🏆</p>

  <ul>
    <li v-for="(a, i) in achievements" :key="i">{{ a }}</li>
  </ul>
</template>
```

</details>

## ✋ Checkpoint

1. What does this render after the button is clicked twice?
   ```vue
   <script setup>
   import { ref } from 'vue'
   const n = ref(1)
   function double() { n.value = n.value * 2 }
   </script>
   <template>
     <button @click="double">{{ n }}</button>
   </template>
   ```
2. This code compiles but the count never changes on screen. Why?
   ```js
   const count = ref(0)
   function add() { count++ }
   ```
3. When would you reach for `v-for`?

<details>
<summary>Answers</summary>

1. `4` - starts at 1, doubles to 2, doubles to 4; the button label tracks `n` automatically.
2. Missing `.value` - it should be `count.value++`. (`count` itself is the ref container, not the number.)
3. Whenever the page should show one element per item of an array - lists, tables, menus, search results.

</details>

## 📚 Further reading

- [Template Syntax - Vue docs](https://vuejs.org/guide/essentials/template-syntax.html) - the full reference for `{{ }}`, `:` and `@`
- [List Rendering - Vue docs](https://vuejs.org/guide/essentials/list.html) - `v-for` details, including why `:key` matters

---

⬅️ [Previous: Your First Project](./02-your-first-project.md) · 🗺️ [Course map](../README.md) · ➡️ Next: [Module 2 · Reactivity](../module-02-reactivity/) *(ask Claude to write it when you're ready!)*
