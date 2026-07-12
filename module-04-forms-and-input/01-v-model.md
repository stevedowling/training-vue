# 1 · v-model - two-way binding on every input type

> **You'll learn:** how to connect form inputs to refs so they stay in sync both ways - and which flavour of `v-model` each input type wants.

## Why this matters

Everything so far displayed data and reacted to clicks; forms are where users *type things in*. Wiring an input by hand means an `@input` handler per field plus a `:value` binding to keep it honest - multiply by ten fields and it's boilerplate soup. `v-model` is both directions in one word, and it's how every form in every Vue app is built.

## The big picture

```vue
<script setup>
import { ref } from 'vue'
const name = ref('')
</script>

<template>
  <input v-model="name" placeholder="Your name">
  <p>Hello, {{ name || 'stranger' }}!</p>
</template>
```

Type in the box: the ref updates *as you type*, and the greeting follows. Set `name.value = 'Steve'` in code: the box updates too. Two-way. Under the hood it's exactly the two bindings you'd write by hand -

```vue
<input :value="name" @input="name = $event.target.value">   <!-- what v-model expands to -->
```

- which means nothing magic is happening: it's Module 1's `:` and `@` in a trenchcoat. Knowing the expansion matters once (lesson 3, when your own components accept v-model); day to day you just write the one word.

## The input-type tour

`v-model` adapts to each input type - same word, correct behaviour:

```vue
<script setup>
import { ref } from 'vue'

const bio = ref('')                 // textarea → string
const wantsNews = ref(false)        // single checkbox → boolean
const toppings = ref([])            // checkbox GROUP → array of the checked values
const size = ref('medium')          // radios → the selected one's value
const country = ref('')             // select → the chosen option's value
</script>

<template>
  <textarea v-model="bio"></textarea>

  <label><input type="checkbox" v-model="wantsNews"> Send me news</label>

  <label><input type="checkbox" value="olives" v-model="toppings"> Olives</label>
  <label><input type="checkbox" value="feta" v-model="toppings"> Feta</label>

  <label><input type="radio" value="small" v-model="size"> Small</label>
  <label><input type="radio" value="large" v-model="size"> Large</label>

  <select v-model="country">
    <option disabled value="">Pick one...</option>
    <option>New Zealand</option>
    <option>Australia</option>
  </select>
</template>
```

The two worth pinning: a **lone checkbox** binds a boolean, but a **group of checkboxes sharing one array ref** collects the checked `value`s into that array - Vue notices the ref is an array and switches behaviour. And radios sharing a ref automatically become a group (no `name` attribute juggling like raw HTML).

## Modifiers: three suffixes you'll actually use

| Modifier | Effect | When |
|---|---|---|
| `v-model.number` | casts input to a number | any numeric field - `<input type="number">` still yields *strings* without it |
| `v-model.trim` | strips surrounding whitespace | names, emails, anything compared or stored |
| `v-model.lazy` | sync on *change* (leaving the field) instead of every keystroke | expensive reactions, or "don't validate while I'm mid-word" |

```vue
<input v-model.number="age" type="number">
<input v-model.trim="email">
```

> [!WARNING]
> The `.number` one bites for real: `qty` from a bare `<input type="number" v-model="qty">` is the string `"5"`, and `"5" + 1` is `"51"`. If a total ever goes weirdly huge, check for a missing `.number` first - it's the forms cousin of Module 2's missing `.value`.

<details>
<summary>🔍 Deep dive: where does the source of truth live now?</summary>

Plain HTML forms keep state in the DOM - the input *is* the data, and JavaScript goes spelunking for it at submit time (`document.querySelector(...).value`). With v-model the ref is the single source of truth and the input is just a *view* of it - the same inversion Module 1 pulled on page updates. Everything downstream falls out of that: live summaries are just template reads, validation is just computeds over refs (next lesson), reset is just reassigning refs, and prefilling an edit form is just initialising refs. You never ask the DOM what the user typed; you already know.

</details>

## 🛠️ Try it - build "Pizza Builder"

New component in the sandbox: `PizzaBuilder.vue` (mount it in `App.vue`, maybe in a `UiCard` - Module 3 pays rent now). One form, every input type:

1. Fields: name (`text` + `.trim`), size (radios: small / medium / large), toppings (checkbox group: at least four options), extra-cheese (single checkbox), delivery notes (textarea), quantity (`number` input + `.number`, minimum 1).
2. A **live order summary** underneath, rebuilt as the user types: "2 large pizzas with olives, feta + extra cheese for Steve". Use a computed for the sentence (Module 2 muscle).
3. A live price too: size sets the base ($8 / $10 / $12), each topping +$1, extra cheese +$2, times quantity. Computed, obviously.
4. Prove `.lazy` to yourself: temporarily make the name field `v-model.lazy` and watch the summary only update when you click away. Decide which feel you prefer, keep that.
5. Break it on purpose: remove `.number` from quantity, set quantity to 2, and watch the price computed misbehave (or not - can you explain why multiplication survives where addition wouldn't?).

<details>
<summary>💡 Hint - the summary computed</summary>

```js
const summary = computed(() => {
  const tops = toppings.value.length ? `with ${toppings.value.join(', ')}` : 'plain'
  const cheese = extraCheese.value ? '+ extra cheese' : ''
  return `${qty.value} ${size.value} pizza${qty.value > 1 ? 's' : ''} ${tops} ${cheese} for ${name.value || 'someone'}`
})
```

</details>

<details>
<summary>✅ Solution - the script and shape</summary>

```vue
<script setup>
import { ref, computed } from 'vue'

const name = ref('')
const size = ref('medium')
const toppings = ref([])
const extraCheese = ref(false)
const notes = ref('')
const qty = ref(1)

const basePrice = computed(() => ({ small: 8, medium: 10, large: 12 }[size.value]))
const price = computed(() =>
  (basePrice.value + toppings.value.length + (extraCheese.value ? 2 : 0)) * qty.value
)
const summary = computed(() => /* hint version */)
</script>
```

Template: the input-type tour, with `v-model.trim="name"` and `v-model.number="qty"`. Step 5's answer: `*` coerces strings to numbers in JavaScript, so multiplication *happens* to survive - but `basePrice + toppings.length` fed a string qty via `+` would concatenate. Fragile luck is not correctness; keep `.number`.

</details>

## ✋ Checkpoint

1. Predict the ref contents: three checkboxes with values `"a"`, `"b"`, `"c"` all bound to `const picks = ref([])`; the user checks c then a. And what would the same clicks produce if `picks` were `ref(false)`?
2. What two attributes/handlers is `<input v-model="q">` shorthand for?
3. A form's total shows `"105"` after adding a $5 item to a $10 base. Diagnose in one sentence.
4. When would you *choose* `.lazy` - name a concrete field.

<details>
<summary>Answers</summary>

1. `['c', 'a']` - the array collects checked values in click order. With a boolean ref, the group degrades to all-toggle-together true/false chaos - the array ref is what makes it a group.
2. `:value="q"` and `@input="q = $event.target.value"`.
3. A string sneaked into `+` - some numeric input is missing `.number`, so 10 + "5" concatenated.
4. Anything where per-keystroke reaction annoys or costs: a field that triggers validation messages (don't shout "invalid email!" at someone mid-typing), or one that fires a search/API call (Module 7 will care a lot).

</details>

## 📚 Further reading

- [Form Input Bindings - Vue docs](https://vuejs.org/guide/essentials/forms.html) - the full tour including multi-selects and dynamic values
- [Vue Playground](https://play.vuejs.org) - the checkpoint predictions take 60 seconds to verify

---

⬅️ [Module home](./README.md) · 🏠 [Course home](../README.md) · ➡️ [Next: Submitting and validating](./02-submitting-and-validating.md)
