# Module 6 · App-wide State

> **You'll learn:** why "props down, events up" stops scaling, how Pinia gives shared state a proper home, and - the part that ages best - the judgment for what belongs in a store versus staying local.

**Builds on:** [Module 3 · Components](../module-03-components/) · Independent of modules 4, 5, 7, take in any order (though [Module 5 · Routing](../module-05-routing/) makes the pain this module fixes much more visceral).

## Lessons

1. [The prop-drilling problem](./01-the-prop-drilling-problem.md) - feel where the pattern breaks, meet the escape hatches, find the insight *(~15 min + exercise)*
2. [Your first Pinia store](./02-your-first-pinia-store.md) - defineStore, using it anywhere, and the storeToRefs gotcha *(~15 min + exercise)*
3. [What goes in the store](./03-what-goes-in-the-store.md) - the three-question flowchart, fat stores / thin components, and the smell field-guide *(~15 min + exercise)*

Take them in order - lesson 1's deliberate suffering is what makes lesson 2 land, and lesson 3 is the judgment that makes both stick.

## When you're done

The cart lives in a store: visible in the header, alive across every page, persistent across reloads, with its rules in one place - plus a second store (theme settings) proving the pattern generalises. Best of all, you can argue *out loud* about what belongs in a store, which is the skill interviews and code reviews actually test.

---

⬅️ [Module 3 · Components](../module-03-components/) · 🏠 [Course home](../README.md) · ➡️ [Start: The prop-drilling problem](./01-the-prop-drilling-problem.md)
