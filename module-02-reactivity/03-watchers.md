# 3 · Watchers - reacting with side effects

> **You'll learn:** how to run your own code when data changes - saving to localStorage, logging, fetching - and how to tell when a watcher is the right tool versus a computed.

## Why this matters

Computeds derive *values*. But some reactions to change aren't values at all - save the game when the score changes, fetch results when the search term changes, log when something goes wrong. Those are **side effects**, and `watch` is Vue's tool for "when this changes, *do* that". It's also the last piece of the reactivity toolkit: after this lesson you have all of it.

## The big picture

```vue
<script setup>
import { ref, watch } from 'vue'

const score = ref(0)

watch(score, (newScore, oldScore) => {
  console.log(`score: ${oldScore} → ${newScore}`)
  localStorage.setItem('score', newScore)      // a side effect - not a derived value
})
</script>
```

Read it as a sentence: *watch `score`; whenever it changes, run this function with the new and old values.* You never call the function yourself - changing `score` anywhere is what calls it.

And the decision that keeps code clean, up front:

| You want... | Tool | Smell test |
|---|---|---|
| a value that follows other values | `computed` | "the total **is**..." |
| an action performed on change | `watch` | "when it changes, **do**..." |

> [!WARNING]
> The classic misuse: `watch(items, () => { total.value = calc() })` - using a watcher to keep a second ref in sync. That's lesson 1's anti-pattern wearing a trenchcoat. If the reaction just produces a value, it's a computed. **Watch is for effects that leave Vue** - storage, network, timers, the console.

## What you can watch

The first argument is flexible - three forms cover practically everything:

```js
watch(score, ...)                          // a ref
watch(() => game.level, ...)               // one property of a reactive object (as a getter)
watch([score, lives], ([s, l]) => ...)     // several sources at once
```

The getter form matters: `watch(game.level, ...)` would pass watch a plain number (dead, lesson 2 style) - wrapping it in `() =>` hands watch something it can re-read and track.

## The two options you'll actually use

**`immediate: true`** - run the callback once right now, then on every change. Perfect when the effect should also happen at startup:

```js
watch(theme, (t) => document.body.className = t, { immediate: true })
// without immediate: applies on the NEXT change only, so the initial theme never paints
```

**`deep: true`** - notice changes *inside* an object or array, not just replacement:

```js
const todos = ref([])
watch(todos, save)                    // fires only if the ARRAY ITSELF is swapped
watch(todos, save, { deep: true })    // fires on push, splice, item edits - what you meant
```

(There's also `once: true` for self-disarming watchers - rarer, nice to know it exists.)

<details>
<summary>🔍 Deep dive: watchEffect - the "just track whatever I use" variant</summary>

`watch`'s sibling skips the explicit source list:

```js
import { watchEffect } from 'vue'

watchEffect(() => {
  document.title = `${appName.value} - score ${score.value}`
})
```

It runs immediately, notices every reactive value the function *read*, and re-runs when any of them change - dependency tracking exactly like a computed, but for an effect. Wonderfully concise for "keep this external thing synced to whatever it needs".

Why the course leads with `watch` anyway: explicit sources make the trigger obvious to the next reader, you get old values (watchEffect can't), and you control the moment with `immediate`. Reach for `watchEffect` when the effect uses several values and you'd only be listing them all twice. One more habit from the pros: both return a stop function - `const stop = watch(...)` then `stop()` - rarely needed inside components (Vue cleans up on unmount automatically), but it's how you'd end a watcher early.

</details>

## 🛠️ Try it - "Click Quest: New Game+"

Module 1's Click Quest has amnesia: refresh the page, progress gone. Fix it with watchers - bring back your Click Quest `App.vue` (or rebuild the small version: `count`, `achievements`, click handler, reset) and:

1. **Persist**: watch `count` and save it with `localStorage.setItem('count', count.value)`. Watch `achievements` too - it's an array, so one of this lesson's options is required. Store it as JSON: `JSON.stringify(achievements.value)`.
2. **Restore**: initialise the refs from storage at startup - `ref(Number(localStorage.getItem('count') ?? 0))` and a JSON.parse equivalent for the array. Refresh mid-game: progress survives. Victory lap achieved.
3. **A real side effect**: watch `count` and set `document.title` to `Clicks: N` - watch the browser tab while you click. Should this one be `immediate`? Decide, then justify.
4. Confirm reset still works *and persists* - click reset, refresh, still zero. (It will be - the watchers catch the reset like any other change. Say *why* in one sentence.)

<details>
<summary>💡 Hint - restoring the array safely</summary>

`JSON.parse(localStorage.getItem('achievements') ?? '[]')` - the `?? '[]'` covers the very first visit when storage is empty. And step 1 needs `{ deep: true }` on the achievements watcher: pushes mutate the array rather than replacing it.

</details>

<details>
<summary>✅ Solution</summary>

```vue
<script setup>
import { ref, watch } from 'vue'

const count = ref(Number(localStorage.getItem('count') ?? 0))
const achievements = ref(JSON.parse(localStorage.getItem('achievements') ?? '[]'))

watch(count, (n) => localStorage.setItem('count', n))
watch(achievements, (a) => localStorage.setItem('achievements', JSON.stringify(a)), { deep: true })
watch(count, (n) => { document.title = `Clicks: ${n}` }, { immediate: true })

function handleClick() {
  count.value++
  if (count.value % 5 === 0) achievements.value.push(`Reached ${count.value}!`)
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
  <ul>
    <li v-for="(a, i) in achievements" :key="i">{{ a }}</li>
  </ul>
</template>
```

Step 3: yes, `immediate` - otherwise the tab title is wrong until the first click after every refresh. Step 4: reset just writes the refs, and watchers don't care *who* changed the data - that's the beauty of the pattern: persistence logic lives in exactly one place, not in every function that touches the state.

</details>

## ✋ Checkpoint

1. Computed or watch? (a) a character count under a text field, (b) auto-saving a draft as it changes, (c) "you're offline" banner state, (d) playing a sound when lives hit zero.
2. Predict: `watch(todos, save)` where `todos = ref([])`, then `todos.value.push('x')` - does `save` run? What single change makes it run?
3. This watcher never fires - why, and what's the fix?
   ```js
   const settings = reactive({ volume: 5 })
   watch(settings.volume, (v) => audio.setVolume(v))
   ```
4. From the exercise: why is persist-in-a-watcher better than calling `localStorage.setItem` inside `handleClick` *and* inside `reset`?

<details>
<summary>Answers</summary>

1. (a) computed - it's a value. (b) watch - storage is a side effect. (c) computed (the *banner text/flag* is derived state; some event updates an `online` ref). (d) watch - a sound is about as side-effecty as effects get.
2. No - push mutates the array without replacing the ref's value. Add `{ deep: true }` (or replace instead of mutate: `todos.value = [...todos.value, 'x']`).
3. `settings.volume` evaluates to the plain number 5 at call time - watch receives a dead value (lesson 2's destructure problem in disguise). Fix: pass a getter, `watch(() => settings.volume, ...)`.
4. One source of truth for the effect: any code that changes the state - today's functions and next month's new ones - gets persistence for free, and there's exactly one place to change the storage format later.

</details>

## 📚 Further reading

- [Watchers - Vue docs](https://vuejs.org/guide/essentials/watchers.html) - includes `watchEffect`, flush timing, and stopping watchers
- [Vue docs: Computed vs Watchers use cases](https://vuejs.org/guide/essentials/computed.html#computed-caching-vs-methods) - the official framing of the same decision table

---

⬅️ [Previous: ref vs reactive](./02-ref-vs-reactive.md) · 🏠 [Course home](../README.md) · ➡️ Next: [Module 3 · Components](../module-03-components/) *(ask Claude to write it when you're ready!)*
