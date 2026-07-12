# 4 · Slots - flexible components

> **You'll learn:** how to leave holes in a component that each caller fills with its own content - the third and final way components talk.

## Why this matters

Props pass *data*, but some components need to receive *markup*: a card that wraps anything, a modal with arbitrary insides, a button whose label might include an icon. Passing HTML through a string prop is misery; slots make it native. They're also how every component library you'll ever use works - after this lesson, their docs read fluently.

## The big picture

A slot is a hole. The child says *where*, the caller says *what*:

```vue
<!-- src/components/UiCard.vue -->
<template>
  <div class="card">
    <slot></slot>              <!-- caller's content lands here -->
  </div>
</template>

<style scoped>
.card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 1rem;
  margin: 0.5rem 0;
}
</style>
```

```vue
<!-- any parent -->
<UiCard>
  <h2>Anything at all</h2>
  <p>Elements, components, {{ bindings }} - all legal here.</p>
</UiCard>

<UiCard>
  <ProductRow v-bind="products[0]" />    <!-- components in the hole? absolutely -->
</UiCard>
```

Same card chrome, wildly different insides - and the card component needs zero knowledge of what it wraps. Compare the prop version (`<UiCard text="...">`, then a `heading` prop, then `headingLevel`, then tears): content is what slots are *for*.

## Fallback content

Whatever sits inside `<slot>` in the child renders **only when the caller provides nothing**:

```vue
<template>
  <button class="fancy">
    <slot>Click me</slot>     <!-- the default label -->
  </button>
</template>
```

```vue
<FancyButton />                      <!-- renders "Click me" -->
<FancyButton>Buy now 🛒</FancyButton>  <!-- caller's label wins -->
```

Cheap to add, and it makes the zero-config case of every component pleasant.

## Named slots: several holes

One hole covers most components; layouts want more. Name the extras:

```vue
<!-- PageSection.vue -->
<template>
  <section>
    <header><slot name="title">Untitled section</slot></header>
    <slot></slot>                                  <!-- the unnamed one is "default" -->
    <footer class="muted"><slot name="footer"></slot></footer>
  </section>
</template>
```

```vue
<PageSection>
  <template #title><h2>🛒 Your Cart</h2></template>

  <ProductRow v-for="p in products" :key="p.id" v-bind="p" />

  <template #footer>3 items - free shipping at $20</template>
</PageSection>
```

`#title` is shorthand for `v-slot:title` (the `@`/`:`/`#` trio is now complete - events, bindings, slots). Content outside any `<template #...>` flows to the default slot. One rule of scope to keep straight: **slot content is compiled in the parent** - it sees the parent's refs and computeds, not the child's. You're mailing finished markup into the hole, not code that runs inside the child.

<details>
<summary>🔍 Deep dive: scoped slots - when the hole needs the child's data</summary>

That scope rule has an escape hatch for the rare inverse case: the *child* has the data, the *parent* decides the markup. The child passes values out through the slot; the parent catches them:

```vue
<!-- child: FilteredList.vue owns the filtering logic -->
<slot v-for="item in filtered" :item="item"></slot>
```

```vue
<!-- parent decides what each item LOOKS like -->
<FilteredList :items="products" #default="{ item }">
  <strong>{{ item.name }}</strong> - ${{ item.price }}
</FilteredList>
```

This is the "renderless component" trick: logic in the child, appearance from the caller - the same separation `useFetch` will achieve with composables in Module 7, in template form. You won't need to *write* one for a while, but library docs (tables, autocompletes, virtual lists) lean on this constantly - now you can read them.

</details>

## Which tool, then?

The module's three channels, on one card:

| The child needs... | Use | Example |
|---|---|---|
| a value | **prop** | `:price="5"` |
| to report something | **emit** | `@add="..."` |
| markup decided by the caller | **slot** | `<UiCard><anything /></UiCard>` |

Undecided between prop and slot for content? If it's a plain string, prop. The moment it might contain markup, structure, or another component - slot.

## 🛠️ Try it - dress the Snack Cart

Finish the module by giving the sandbox some chrome:

1. Build `UiCard.vue` as above. Wrap each major area of `App.vue` in one (the product list, the totals, the Click Quest if it still lives there).
2. Add a named `title` slot to UiCard with fallback content `"Card"`. Give each card a real title - at least one with an emoji and at least one relying on the fallback (to see it work).
3. Build `FancyButton.vue`: styled button, default slot for the label with a fallback, and - combining the whole module - it should still work as a button: no `@click` handling inside; the parent's `@click` on the tag just works. Replace two ordinary buttons with it. (Surprise you get for free: native events on a single-root component fall through to its root element.)
4. The reusability test: drop a `<UiCard>` with completely unrelated content - a haiku, a todo list, anything. Zero changes to UiCard allowed. If it just works, slots have done their job.

<details>
<summary>💡 Hint - UiCard with the title slot</summary>

```vue
<template>
  <div class="card">
    <h3><slot name="title">Card</slot></h3>
    <slot></slot>
  </div>
</template>
```

Callers: `<UiCard><template #title>🛒 Cart</template> ...content... </UiCard>`

</details>

<details>
<summary>✅ Solution - the pieces</summary>

```vue
<!-- FancyButton.vue -->
<template>
  <button class="fancy"><slot>Click me</slot></button>
</template>

<style scoped>
.fancy {
  background: #42b883; color: white;
  border: none; border-radius: 6px;
  padding: 0.4rem 1rem; cursor: pointer;
}
</style>
```

```vue
<!-- usage in App.vue -->
<UiCard>
  <template #title>🛒 Your Cart</template>
  <ProductRow v-for="p in products" :key="p.id" v-bind="p" :qty="qty[p.id] ?? 0"
    @add="addToCart" @remove="removeFromCart" />
</UiCard>

<UiCard>   <!-- title falls back to "Card" - fine for the demo box -->
  <p>{{ itemCount }} items - ${{ total }}</p>
  <FancyButton @click="happyHour">Happy hour ☀️</FancyButton>
</UiCard>
```

</details>

## ✋ Checkpoint

1. Prop or slot? (a) a card's border colour, (b) a card's body content, (c) a button's disabled state, (d) a modal's footer buttons.
2. Predict: `<UiCard />` self-closing, where UiCard's title slot has fallback "Card" and its default slot has no fallback. What renders?
3. This doesn't compile the way its author hoped - the child has `todos` but the parent doesn't. Why does it fail, per the scope rule?
   ```vue
   <TodoPanel>
     <p>{{ todos.length }} items</p>
   </TodoPanel>
   ```
4. Name the three component channels and give each a four-word job description.

<details>
<summary>Answers</summary>

1. (a) prop - it's a value. (b) slot - it's markup. (c) prop - a boolean. (d) slot - buttons are structure, and different modals want different ones.
2. The card chrome with heading "Card" (fallback fires) and an empty body - missing slot content simply renders nothing.
3. Slot content compiles in the *parent's* scope - `todos` must exist in the parent. Fixes: pass it to the parent, or use a scoped slot so the child hands `todos` out through the hole.
4. Props: data flows down. Emits: events flow up. Slots: markup flows in.

</details>

## 📚 Further reading

- [Slots - Vue docs](https://vuejs.org/guide/components/slots.html) - named, scoped, and the compile-scope rules
- [Fallthrough Attributes - Vue docs](https://vuejs.org/guide/components/attrs.html) - why FancyButton's `@click` "just worked" in step 3

---

⬅️ [Previous: Emits](./03-emits.md) · 🏠 [Course home](../README.md) · ➡️ Next: [Module 4 · Forms & Input](../module-04-forms-and-input/)
