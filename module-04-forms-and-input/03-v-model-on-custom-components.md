# 3 · v-model on custom components

> **You'll learn:** how to make your *own* components work with `v-model` - so a star rating, a toggle, or a whole address block binds as easily as an `<input>`.

## Why this matters

Real forms outgrow native inputs fast: star ratings, tag pickers, toggles, quantity steppers. Without this lesson each one needs a prop *and* an event *and* wiring at every call site; with it, callers just write `v-model="rating"` and your component feels built-in. This is also the lesson where Module 3 and Module 4 snap together into one mental model.

## The big picture

`defineModel` makes a component v-model-able in one line:

```vue
<!-- src/components/StarRating.vue -->
<script setup>
const model = defineModel()          // the two-way value, as a ref
</script>

<template>
  <span>
    <button v-for="n in 5" :key="n" @click="model = n">
      {{ n <= model ? '★' : '☆' }}
    </button>
  </span>
</template>
```

```vue
<!-- any parent -->
<script setup>
import { ref } from 'vue'
const rating = ref(0)
</script>

<template>
  <StarRating v-model="rating" />
  <p>You rated us {{ rating }}/5</p>
</template>
```

Inside the child, `model` behaves like a ref you already know: read it, assign to it (`.value` in script; templates unwrap as usual). Assignments don't *really* mutate anything locally - they flow to the parent's `rating`, which flows back down. Two-way, and the parent still owns the data.

## What's actually happening (no magic allowed)

Lesson 1 showed `v-model` on an `<input>` expands to a `:value` bind plus an `@input` handler. On a *component*, it expands to a prop plus an event:

```vue
<StarRating v-model="rating" />
<!-- is exactly: -->
<StarRating :modelValue="rating" @update:modelValue="rating = $event" />
```

So a v-model-able component is nothing new - it's a component with a `modelValue` prop and an `update:modelValue` emit, which you could have built with Module 3 alone:

```js
// the manual version - what defineModel generates for you
const props = defineProps({ modelValue: Number })
const emit = defineEmits(['update:modelValue'])
// ...then emit('update:modelValue', n) instead of assigning
```

`defineModel` wraps that pair in a ref-shaped bow. Props down, events up - the pattern didn't bend; it got a shorthand. You'll still meet the manual spelling in older codebases and library sources, so recognising it is part of the lesson.

## Options and multiple models

`defineModel` accepts the same treatment as any prop, and components can expose several models by name:

```js
const model = defineModel({ type: Number, default: 0 })   // typed, defaulted
```

```vue
<!-- a range picker with two bound values -->
<script setup>
const min = defineModel('min')
const max = defineModel('max')
</script>
```

```vue
<PriceRange v-model:min="lowest" v-model:max="highest" />
```

> [!TIP]
> Reach for a custom v-model when your component's whole *job* is editing one value - ratings, toggles, steppers, pickers. If it edits several unrelated things or mainly *announces actions*, plain props + named emits stay clearer. "Would a native input do this job if one existed?" is the test.

<details>
<summary>🔍 Deep dive: v-model modifiers on your components</summary>

Native v-model has `.trim` and friends; your components can accept custom modifiers too. `defineModel` hands you what the caller wrote:

```js
const [model, modifiers] = defineModel({
  set(value) {
    return modifiers.capitalize
      ? value.charAt(0).toUpperCase() + value.slice(1)
      : value
  }
})
```

```vue
<NameInput v-model.capitalize="firstName" />
```

The `set` transform runs on every write before it reaches the parent - the same hook a writable computed gave you in Module 2. Niche, but it's how polished libraries make `v-model.something` feel native, and now the syntax won't mystify you.

</details>

## 🛠️ Try it - "Rate your pizza"

Extend the checkout flow with two custom inputs:

1. Build `StarRating.vue` with `defineModel({ type: Number, default: 0 })`. Stars fill on hover-free click (as in the lesson); add a small "(clear)" button that sets it back to 0.
2. Build `QtyStepper.vue` - the classic − / count / + control - with a typed model and `min`/`max` *props* (default 1 and 10). Clamp inside the component: + past max does nothing. Swap it in for PizzaBuilder's raw quantity input, replacing the `.number` worry entirely.
3. Add StarRating to the *success* card ("How was ordering? ★★★☆☆") bound to a `rating` ref, and show a thank-you line once rating > 0.
4. Prove the expansion to yourself: temporarily replace `v-model="rating"` with the longhand `:modelValue` + `@update:modelValue` version. Identical behaviour? Revert to the short form, permanently wiser.
5. Component-quality pass: both new components should work dropped into *any* app - no imports from your sandbox, no knowledge of pizzas. That's the reusability bar custom inputs must clear.

<details>
<summary>💡 Hint - the stepper's clamp</summary>

```js
const model = defineModel({ type: Number, default: 1 })
const props = defineProps({ min: { type: Number, default: 1 }, max: { type: Number, default: 10 } })

function step(delta) {
  const next = model.value + delta
  if (next >= props.min && next <= props.max) model.value = next
}
```

defineModel and defineProps coexist happily - the model is just one more prop underneath.

</details>

<details>
<summary>✅ Solution - StarRating.vue</summary>

```vue
<script setup>
const model = defineModel({ type: Number, default: 0 })
</script>

<template>
  <span class="stars">
    <button v-for="n in 5" :key="n" @click="model = n" :aria-label="`Rate ${n}`">
      {{ n <= model ? '★' : '☆' }}
    </button>
    <button v-if="model > 0" class="clear" @click="model = 0">(clear)</button>
  </span>
</template>

<style scoped>
.stars button { border: none; background: none; font-size: 1.4rem; cursor: pointer; }
.clear { font-size: 0.8rem; color: #888; }
</style>
```

Usage: `<StarRating v-model="rating" />` - and in the parent, `rating` is a plain ref that some *other* card can also read (`{{ rating }}/5`), because the parent still owns it.

</details>

## ✋ Checkpoint

1. Write the longhand expansion of `<Toggle v-model="dark" />`.
2. Inside a `defineModel` component, `model.value = 5` runs - where does the 5 actually *live* a millisecond later, and how did it get there?
3. A `SearchBox` component emits what the user typed. Should it be `v-model` or a `@search` emit - and what's the question you ask to decide?
4. Predict: parent binds `v-model="qty"` where `qty = ref(1)`, child's stepper is clamped to max 10, parent code later sets `qty.value = 99`. What does the stepper display, and who was supposed to enforce the rule?

<details>
<summary>Answers</summary>

1. `<Toggle :modelValue="dark" @update:modelValue="dark = $event" />`
2. In the parent's ref - the assignment became an `update:modelValue` emit; the parent updated its own state, which flowed back down the prop. The child holds nothing.
3. Is the component's job to *edit a value* the parent keeps (v-model - e.g. a live filter string), or to *announce an action* (emit - e.g. "user pressed search, go query")? Either can be right for a search box; the deciding question is what the parent does with it.
4. It displays 99 - the clamp lives in the child's `step()`, which parent writes bypass entirely. Rules the parent must obey belong in the parent (or validated there too) - the child's clamp is UX, not law. Module 4's forms lesson said the same about servers; ownership arguments repeat at every scale.

</details>

## 📚 Further reading

- [Component v-model - Vue docs](https://vuejs.org/guide/components/v-model.html) - defineModel, multiple models, modifiers
- [Slots + v-model together](https://vuejs.org/examples/#form-bindings) - the official examples gallery; read a few forms and notice you can now explain every line

---

⬅️ [Previous: Submitting and validating](./02-submitting-and-validating.md) · 🏠 [Course home](../README.md) · ➡️ Next: [Module 5 · Routing](../module-05-routing/)
