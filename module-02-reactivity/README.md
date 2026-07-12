# Module 2 · Reactivity

> **You'll learn:** the full reactivity toolkit - values that derive themselves with `computed`, what `ref` really is (and when `reactive` is the alternative), and side effects with `watch`. This is Vue's core superpower; every later module spends it.

**Builds on:** [Module 1 · First Steps](../module-01-first-steps/)

## Lessons

1. [Computed Properties](./01-computed-properties.md) - values that calculate themselves, and the anti-pattern they replace *(~15 min + exercise)*
2. [ref vs reactive](./02-ref-vs-reactive.md) - why `.value` exists, and the three ways reactivity silently breaks *(~15 min + exercise)*
3. [Watchers](./03-watchers.md) - reacting to change with side effects, and computed-or-watch judgment *(~15 min + exercise)*

Take them in order - lesson 2 explains machinery lesson 1 relies on, and lesson 3's judgment calls lean on both.

## When you're done

Your Click Quest game survives a page refresh (localStorage via watchers), your Snack Cart's math can never be stale (computeds), and "the page stopped updating" bugs hold no fear - you'll recognise all three causes on sight.

---

⬅️ [Module 1 · First Steps](../module-01-first-steps/) · 🏠 [Course home](../README.md) · ➡️ [Start: Computed Properties](./01-computed-properties.md)
