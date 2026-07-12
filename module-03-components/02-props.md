# 2 · Props - passing data down

> **You'll learn:** how a parent hands data to a child component - turning yesterday's identical copies into one component, many configurations.

## Why this matters

Lesson 1 ended at a wall: `ProductRow` couldn't know which product it was. Props are the answer, and they're half of the pattern that structures every Vue app ever built. A component with props stops being a photocopy and becomes a *template with blanks* - same structure, different data each use.

## The big picture

The child declares what it accepts; the parent fills in the blanks per use:

```vue
<!-- src/components/ProductRow.vue -->
<script setup>
const props = defineProps({
  name: String,
  price: Number,
  emoji: String
})
</script>

<template>
  <p>{{ emoji }} {{ name }} - ${{ price }}</p>
</template>
```

```vue
<!-- App.vue -->
<template>
  <ProductRow name="Coffee" :price="5" emoji="☕" />
  <ProductRow name="Cookie" :price="3" emoji="🍪" />
</template>
```

One component, two products. Add a third `<ProductRow>` and there's no new code to write - that's the payoff, and it compounds for the rest of your Vue career.

## Declaring props

`defineProps` takes an object of name → type (available in `<script setup>` without importing). Types can carry more detail when it earns its keep:

```js
const props = defineProps({
  name: { type: String, required: true },     // forget it → console warning
  price: { type: Number, required: true },
  emoji: { type: String, default: '🛒' }      // optional, with a fallback
})
```

In the template, use props like refs: `{{ name }}`. In script, they live on the returned object: `props.name` (no `.value` - props aren't refs, but they *are* reactive: when the parent's data changes, the child re-renders).

## Static vs bound: the colon, again

Module 1's deep dive introduced `:` as "make this attribute dynamic" - with props it graduates to load-bearing:

```vue
<ProductRow name="Coffee" />      <!-- the literal string "Coffee" -->
<ProductRow :name="topSeller" />  <!-- whatever the variable holds -->
<ProductRow :price="5" />         <!-- the NUMBER 5 -->
<ProductRow price="5" />          <!-- ❌ the STRING "5" - type warning, math bugs -->
```

That last pair is the classic slip: **anything that isn't a string needs the colon**, even a hardcoded number or boolean - without it you're passing the characters, not the value.

Props and `v-for` were made for each other - the data-driven list:

```vue
<script setup>
import { ref } from 'vue'
import ProductRow from './components/ProductRow.vue'

const products = ref([
  { id: 1, name: 'Coffee', price: 5, emoji: '☕' },
  { id: 2, name: 'Cookie', price: 3, emoji: '🍪' },
  { id: 3, name: 'Toastie', price: 8, emoji: '🥪' },
])
</script>

<template>
  <ProductRow
    v-for="p in products"
    :key="p.id"
    :name="p.name"
    :price="p.price"
    :emoji="p.emoji"
  />
</template>
```

Now the *data* is the page: add an object to the array, a row appears. (Also note `:key` finally has real IDs to use instead of the index - stable identity, like the Module 1 warning wanted.)

## One-way street

Props flow down, and only down. The child must never write to them:

```js
const props = defineProps({ price: Number })
props.price = 99   // ❌ warning - and the parent would overwrite it on next render anyway
```

Why so strict? Because if children could edit props, a bug in *any* component could corrupt data *anywhere*, and you could never tell where a value came from. One-way flow keeps every piece of data owned by exactly one component - to change it, ask the owner. "Ask the owner" is precisely what **emits** are, next lesson.

> [!TIP]
> Legitimately need to start from a prop and diverge? Copy it into local state once (`const draft = ref(props.name)`) or derive with a computed (`const label = computed(() => props.name.toUpperCase())`). Both respect the one-way street: the original stays the parent's.

<details>
<summary>🔍 Deep dive: shorter spellings you'll meet in the wild</summary>

Three variations appear constantly in real codebases:

```vue
<ProductRow v-bind="p" />          <!-- object spread: passes every key of p as a prop -->
<ProductRow :name />               <!-- shorthand when variable and prop share a name (Vue 3.4+) -->
```

```js
const { name, price } = defineProps({ ... })   // destructured props keep reactivity
                                               // in Vue 3.5+ - older guides warn against
                                               // this; modern Vue made it safe
```

All sugar over what you just learned. Recognise them, adopt them when they read better - `v-bind="p"` especially turns the six-line ProductRow call into one.

</details>

## 🛠️ Try it - Snack Cart, componentized (part 1)

Upgrade lesson 1's static `ProductRow` into the real thing:

1. Move the product data into an array of objects in `App.vue` (id, name, price, emoji - three products minimum, invent a new snack).
2. Give `ProductRow.vue` proper `defineProps` - name and price `required: true`, emoji with a default. Render one line per product via `v-for`.
3. Prove the types matter: pass `price="5"` (no colon) on purpose, open the console, read the warning, fix it. This warning will save you an hour someday - meet it on your own terms.
4. Prove reactivity flows down: add a "Happy hour ☀️" button in `App.vue` that halves every price in the array. Rows update - no child code changed.
5. The wall, again (on purpose): put the + / - quantity buttons *inside* `ProductRow`. Try to make them work by changing a `qty` prop. Read the warning, then leave a `<!-- TODO: emits, next lesson -->`. (Feeling *why* the one-way street exists beats being told.)

<details>
<summary>💡 Hint - happy hour</summary>

```js
function happyHour() {
  products.value = products.value.map(p => ({ ...p, price: p.price / 2 }))
}
```

Replacing the array through `.value` is ref-friendly (Module 2, lesson 2) and the props carry the new prices down automatically.

</details>

<details>
<summary>✅ Solution - ProductRow.vue</summary>

```vue
<script setup>
defineProps({
  name: { type: String, required: true },
  price: { type: Number, required: true },
  emoji: { type: String, default: '🛒' }
})
</script>

<template>
  <p>{{ emoji }} {{ name }} - ${{ price }}</p>
</template>
```

With App.vue rendering `<ProductRow v-for="p in products" :key="p.id" :name="p.name" :price="p.price" :emoji="p.emoji" />` (or the `v-bind="p"` shorthand if you read the deep dive - it passes the extra `id` as an unused prop, which is harmless here).

</details>

## ✋ Checkpoint

1. Predict the rendering and the console for `<ProductRow :name="'Tea'" price="4" />` given lesson's ProductRow with `price: { type: Number, required: true }`.
2. A child component needs to display a price with 20% tax added. Prop in `price` - then what, without breaking the one-way street?
3. Why does `<ProductRow enabled />`... wait, no colon, no value - what does the child's `enabled: Boolean` prop receive? (This one's a gift from HTML.)
4. In one sentence: why are props read-only?

<details>
<summary>Answers</summary>

1. It renders ("4" displays fine) but the console warns: expected Number, got String - the missing colon passed characters, not a number. `:price="4"` fixes it.
2. `const withTax = computed(() => props.price * 1.2)` - derive, display the computed, never touch the prop.
3. `true` - a bare attribute on a Boolean prop means true, mirroring HTML's `<input disabled>`. The one place the no-colon rule bends.
4. So every piece of data has exactly one owner - the component that declared the ref - making "who changed this?" always answerable.

</details>

## 📚 Further reading

- [Props - Vue docs](https://vuejs.org/guide/components/props.html) - declaration styles, validation, and the full casing rules (camelCase props become kebab-case attributes)
- [One-way data flow - Vue docs](https://vuejs.org/guide/components/props.html#one-way-data-flow) - the official version of the one-way street argument

---

⬅️ [Previous: Your First Component](./01-your-first-component.md) · 🏠 [Course home](../README.md) · ➡️ [Next: Emits](./03-emits.md)
