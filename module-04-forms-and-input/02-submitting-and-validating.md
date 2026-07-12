# 2 · Submitting and validating a form

> **You'll learn:** how to handle a form's submit without the page reloading, validate input reactively, and show error messages that help instead of nag.

## Why this matters

A form that binds inputs is half a form. The other half is the moment of truth: the user hits Submit, and your app must catch the event, judge the data, refuse it kindly or accept it visibly. Do this well and forms feel effortless; do it badly and users meet the two classic disasters - the page-reload that eats their input, and the red error yelling at them mid-word.

## The big picture

```vue
<script setup>
import { ref, computed } from 'vue'

const email = ref('')
const submitted = ref(false)

const emailError = computed(() => {
  if (!email.value) return 'Email is required'
  if (!email.value.includes('@')) return 'That doesn\'t look like an email'
  return null                                        // null = valid
})
const formValid = computed(() => !emailError.value)

function onSubmit() {
  if (!formValid.value) return
  submitted.value = true                             // later: send it somewhere real
}
</script>

<template>
  <form @submit.prevent="onSubmit">
    <input v-model.trim="email" placeholder="you@example.com">
    <p v-if="emailError" class="error">{{ emailError }}</p>

    <button :disabled="!formValid">Sign up</button>
  </form>
  <p v-if="submitted">🎉 Welcome aboard!</p>
</template>
```

Everything in this module so far, plus two newcomers: `@submit.prevent` on the form, and validation as *computeds*. Those two ideas are the whole lesson - the rest is craft.

## @submit.prevent: stopping the time machine

HTML forms predate JavaScript apps: their native submit *navigates to a new page*, wiping your app's state like it's 1996. `.prevent` is an **event modifier** that calls `event.preventDefault()` for you:

```vue
<form @submit.prevent="onSubmit">
```

Two habits that come with it:

- **Handle submit on the `<form>`, not click on the button.** Form submit fires for Enter-in-a-field too, and keyboards are how fast users live. A real `<button>` inside a form triggers submit automatically.
- `.prevent` has cousins you'll meet (`@click.stop`, `@keyup.enter`, `@keyup.esc`) - same idea, DOM fiddling compressed into template syntax.

## Validation is just computeds

No library needed for most forms - a rule is a computed that returns an error string or null:

```js
const nameError = computed(() => {
  if (!name.value) return 'Name is required'
  if (name.value.length < 2) return 'A bit longer, surely'
  return null
})
const ageError = computed(() => {
  if (age.value === '' || age.value === null) return 'Age is required'
  if (age.value < 13) return 'You must be 13 or older'
  return null
})
const formValid = computed(() => !nameError.value && !ageError.value)
```

Because they're computeds, errors update live as the user types, the submit button's `:disabled="!formValid"` can never disagree with the messages, and there's no "run the validators" step to forget - Module 2's derive-don't-sync principle, wearing a form.

## The politeness layer

Live validation has a rudeness problem: "Email is required" glaring at a pristine, untouched form. The standard fix is to *hold* errors until the field has been visited - track "touched" per field with `@blur` (the losing-focus event):

```js
const touched = ref({ email: false })
const showEmailError = computed(() => touched.value.email && emailError.value)
```

```vue
<input v-model.trim="email" @blur="touched.email = true">
<p v-if="showEmailError" class="error">{{ showEmailError }}</p>
```

Validity is always *computed* (the button stays properly disabled); only the *display* waits for politeness. A second common courtesy: on a submit attempt, mark everything touched - the user asked for judgment, deliver all of it.

> [!TIP]
> Rule of thumb for tone: errors appear on blur or submit, *disappear* the instant typing fixes them. Quick to forgive, slow to accuse - computeds give you the quick-to-forgive half for free.

<details>
<summary>🔍 Deep dive: when to reach for a form library</summary>

Hand-rolled computeds scale happily to five or eight fields. Past that - multi-step wizards, server-driven rules, async checks ("is this username taken?"), field arrays - the bookkeeping (touched flags, per-field wiring) gets repetitive, and libraries like **VeeValidate** or **@tanstack/vue-form** earn their install, usually paired with a schema validator like **Zod** ("this form's data must look like this shape"). The concepts transfer exactly: rules are still functions over values, errors still render conditionally, submit still gates on validity. Learn the hand-rolled way first (you are), and the libraries read as "our computeds, industrialized". One thing to keep either way: **client-side validation is UX, not security** - whatever receives the data must validate again on the server, because browsers are politely asked, never trusted.

</details>

## 🛠️ Try it - "Pizza Builder: checkout"

Give lesson 1's PizzaBuilder a real submit:

1. Add an email field (`.trim`). Rules: name required (2+ chars), email required + contains `@`, quantity 1-10. Each rule a computed returning message-or-null, plus a `formValid`.
2. Wrap the fields in a `<form @submit.prevent="placeOrder">` with a FancyButton (Module 3!) as the submit button, `:disabled="!formValid"`.
3. Politeness: errors show only after their field's `@blur`, and `placeOrder` marks all fields touched (so a premature Enter shows everything at once). Style `.error` red but *calm*.
4. On valid submit: hide the form (`v-if`), show a success card with the lesson-1 order summary and price, plus a "Start another order" button that resets every ref (write a `resetForm()` - and notice how a reset is trivial when refs are the source of truth).
5. Torture test, two minutes: submit empty via Enter; type an email, delete it; enter quantity 99. Every path should respond sensibly and no path should reload the page.

<details>
<summary>💡 Hint - marking everything touched on submit</summary>

```js
function placeOrder() {
  Object.keys(touched.value).forEach(k => touched.value[k] = true)
  if (!formValid.value) return
  orderPlaced.value = true
}
```

</details>

<details>
<summary>✅ Solution - the validation core</summary>

```js
const touched = ref({ name: false, email: false, qty: false })
const orderPlaced = ref(false)

const nameError = computed(() => {
  if (!name.value) return 'Name is required'
  if (name.value.length < 2) return 'At least 2 characters'
  return null
})
const emailError = computed(() => {
  if (!email.value) return 'Email is required'
  if (!email.value.includes('@')) return 'That doesn\'t look like an email'
  return null
})
const qtyError = computed(() => {
  if (qty.value < 1) return 'At least one pizza - you\'re here, after all'
  if (qty.value > 10) return 'Max 10 per order'
  return null
})
const formValid = computed(() => !nameError.value && !emailError.value && !qtyError.value)

function resetForm() {
  name.value = ''; email.value = ''; qty.value = 1
  toppings.value = []; extraCheese.value = false; notes.value = ''; size.value = 'medium'
  touched.value = { name: false, email: false, qty: false }
  orderPlaced.value = false
}
```

Template skeleton: `<form @submit.prevent="placeOrder" v-if="!orderPlaced">` ... `<UiCard v-else>` with the summary and `<FancyButton @click="resetForm">Start another order</FancyButton>`.

</details>

## ✋ Checkpoint

1. What two distinct things does `@submit.prevent="onSubmit"` do?
2. Predict: submit button has `:disabled="!formValid"`, user presses Enter in a text field while the form is invalid. Does `onSubmit` run?
3. A code review finds `function validate() { errors.value = runRules() }` called at the end of six different handlers. Which module-2 lesson is this, and what's the fix?
4. Why does "touched" gate the error *display* but not the `formValid` computed?

<details>
<summary>Answers</summary>

1. Registers your handler for the form's submit event *and* cancels the browser's native submit-and-navigate behaviour.
2. No - a disabled submit button means the form has no enabled submitter, so Enter won't dispatch submit in practice. (Belt and braces: the guard clause in onSubmit protects the paths that can still reach it.)
3. The sync-by-hand anti-pattern - `errors` wants to be a computed so it can never be stale and the six call sites vanish.
4. Validity is a fact about the data (the button must respect it from moment zero); showing the message is a courtesy about *timing*. Facts compute always; courtesies wait for blur.

</details>

## 📚 Further reading

- [Event Handling: modifiers - Vue docs](https://vuejs.org/guide/essentials/event-handling.html#event-modifiers) - the full `.prevent`/`.stop`/key-modifier family
- [VeeValidate](https://vee-validate.logaretm.com/v4/) - bookmark for the day a form outgrows hand-rolled

---

⬅️ [Previous: v-model](./01-v-model.md) · 🏠 [Course home](../README.md) · ➡️ [Next: v-model on custom components](./03-v-model-on-custom-components.md)
