# 1 · Computed Properties

> **You'll learn:** how to create values that calculate themselves from other values - and never write "update the total" code again.

## Why this matters

Real apps are full of *derived* values: a cart total derived from items, a filtered list derived from a search term, an "is the form valid?" flag derived from its fields. Deriving them by hand means updating them in every place the source data changes - miss one and the UI lies. `computed` makes the derivation automatic, exactly like reactivity made display automatic in Module 1.

## The big picture

A `computed` is a value defined by a *formula*. Change any ingredient, the result updates itself:

```vue
<script setup>
import { ref, computed } from 'vue'

const price = ref(4)
const quantity = ref(2)

const total = computed(() => price.value * quantity.value)  // the formula
</script>

<template>
  <p>{{ quantity }} × ${{ price }} = ${{ total }}</p>
  <button @click="quantity++">More</button>   <!-- total updates itself -->
</template>
```

No code ever *sets* `total`. It can't be set - it's defined by its formula, the way a spreadsheet cell is defined by `=A1*B1`. That spreadsheet analogy is genuinely how to think about it: refs are cells you type numbers into, computeds are cells with formulas.

## Using computed

```js
const total = computed(() => price.value * quantity.value)

console.log(total.value)    // read with .value in script, like a ref
// total.value = 99         // ❌ error - you don't set a formula cell
```

A computed behaves like a read-only ref: `.value` in script, automatic unwrapping in templates. Anything reactive read inside the formula becomes a dependency - Vue tracks them for you:

```js
const todos = ref(['Learn computed', 'Do the exercise'])

const todoCount = computed(() => todos.value.length)
const anyLeft = computed(() => todoCount.value > 0)        // computeds can use computeds
const shoutyTodos = computed(() =>
  todos.value.map(t => t.toUpperCase())                     // derive arrays, not just numbers
)
```

## Why not just a function?

You could write `function total() { return price.value * quantity.value }` and call `{{ total() }}` in the template. It works - but computed has two wins:

| | `computed` | function call |
|---|---|---|
| Recalculates | only when a dependency changed | every single re-render |
| Result is | **cached** between changes | thrown away each time |
| Reads as | a value: `{{ total }}` | an action: `{{ total() }}` |

The caching matters when the formula is expensive (filtering 5,000 rows), but the *readability* rule is the one to keep: **values you look at are computeds, things you do are functions.**

## The anti-pattern computed replaces

Watch for this shape in your own code - two refs you have to keep in sync by hand:

```js
// ❌ fragile: every place that touches items must also fix total
const items = ref([])
const total = ref(0)

function addItem(item) {
  items.value.push(item)
  total.value += item.price     // forget this line anywhere = wrong UI
}
```

```js
// ✅ robust: total cannot be wrong
const items = ref([])
const total = computed(() => items.value.reduce((sum, i) => sum + i.price, 0))
```

> [!TIP]
> The smell to notice: a ref that is never set *directly by the user*, only recalculated after other changes. That ref wants to be a computed. **If it can be derived, derive it.**

<details>
<summary>🔍 Deep dive: writable computeds exist (and when they're worth it)</summary>

A computed *can* accept writes if you give it both directions:

```js
const firstName = ref('Grace')
const lastName = ref('Hopper')

const fullName = computed({
  get: () => `${firstName.value} ${lastName.value}`,
  set: (val) => { [firstName.value, lastName.value] = val.split(' ') }
})

fullName.value = 'Ada Lovelace'   // writes back through the setter
```

Useful occasionally (Module 4 meets a case with `v-model`), but rare - if you find yourself writing setters often, the data model probably wants restructuring instead. File it under "recognise it when you see it".

</details>

## 🛠️ Try it - build "Snack Cart"

In `vue-sandbox`, replace `App.vue` with a tiny shop. Two products (say, coffee $5 and cookie $3), each with **+ / -** buttons for quantity, then let computeds do all the math:

- Make these three real computeds: `itemCount` (total items), `total` (dollars), and `freeShipping` (true when total ≥ $20)
- Show "🚚 Free shipping!" with `v-if="freeShipping"` (Module 1 skills stay warm)
- A "Clear cart" button that only resets the two quantity refs - if your computeds are right, everything else fixes itself. That's the test.
- Guard rail: don't let a quantity go below 0

<details>
<summary>💡 Hint - the shape of the data</summary>

Simplest version: two refs, `coffeeQty` and `cookieQty`. Then `itemCount = computed(() => coffeeQty.value + cookieQty.value)` and total follows the same pattern with prices. (An array of items + `reduce` is the fancy version - both are correct.)

</details>

<details>
<summary>✅ Solution</summary>

```vue
<script setup>
import { ref, computed } from 'vue'

const coffeeQty = ref(0)
const cookieQty = ref(0)

const itemCount = computed(() => coffeeQty.value + cookieQty.value)
const total = computed(() => coffeeQty.value * 5 + cookieQty.value * 3)
const freeShipping = computed(() => total.value >= 20)

function clear() {
  coffeeQty.value = 0
  cookieQty.value = 0
}
</script>

<template>
  <h1>Snack Cart</h1>

  <p>
    ☕ Coffee ($5) ×{{ coffeeQty }}
    <button @click="coffeeQty++">+</button>
    <button @click="coffeeQty > 0 && coffeeQty--">-</button>
  </p>
  <p>
    🍪 Cookie ($3) ×{{ cookieQty }}
    <button @click="cookieQty++">+</button>
    <button @click="cookieQty > 0 && cookieQty--">-</button>
  </p>

  <p>{{ itemCount }} items - ${{ total }}</p>
  <p v-if="freeShipping">🚚 Free shipping!</p>

  <button @click="clear">Clear cart</button>
</template>
```

Note what `clear()` touches: only the two source refs. Count, total, and the shipping banner all correct themselves - that's the whole lesson in one function.

</details>

## ✋ Checkpoint

1. What does this show after the button is clicked once?
   ```js
   const n = ref(2)
   const double = computed(() => n.value * 2)
   ```
   ```html
   <button @click="n++">{{ double }}</button>
   ```
2. This code throws an error. Why?
   ```js
   const total = computed(() => price.value * qty.value)
   total.value = 0
   ```
3. Your app has `const isValid = ref(false)` and four different functions each end with `isValid.value = checkForm()`. What's the refactor?

<details>
<summary>Answers</summary>

1. `6` - n becomes 3, and the computed re-derives to 6 on its own.
2. Computeds are read-only (no setter was defined) - they're formula cells, not data cells. Reset the *sources* instead.
3. `const isValid = computed(() => checkForm())` - then delete those four assignment lines; it can never be stale again.

</details>

## 📚 Further reading

- [Computed Properties - Vue docs](https://vuejs.org/guide/essentials/computed.html) - includes the caching details and the writable form
- [Vue Playground](https://play.vuejs.org) - paste the checkpoint snippets in and test your predictions

---

⬅️ [Module home](./README.md) · 🏠 [Course home](../README.md) · ➡️ [Next: ref vs reactive](./02-ref-vs-reactive.md)
