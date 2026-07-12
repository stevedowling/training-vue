# Module 7 · Talking to APIs

> **You'll learn:** how Vue apps get real data - fetch + async/await into refs, the loading/error/success pattern that makes pages honest, and composables, Vue's flagship trick for packaging logic you'll reuse forever.

**Builds on:** [Module 2 · Reactivity](../module-02-reactivity/) · Independent of modules 4-6, take in any order (exercises use the router from [Module 5](../module-05-routing/) for new pages - adaptable if you haven't done it yet).

## Lessons

1. [Fetching data](./01-fetching-data.md) - fetch, async/await, onMounted, and reading an API's response shape *(~15 min + exercise)*
2. [Loading, error, success](./02-loading-error-success.md) - the three-state pattern, the res.ok trap, and deliberate sabotage testing *(~15 min + exercise)*
3. [Composables](./03-composables.md) - extracting useFetch, search-as-you-type, and where VueUse fits *(~20 min + exercise)*

Take them in order - each lesson's code becomes the next one's raw material, ending with a composable you'll genuinely reuse.

## When you're done

Your app shows real products from a real API, behaves honestly on slow and broken networks (you'll have sabotaged it yourself to prove it), and its data-fetching lives in a `useFetch` composable with a debounced live search to show for it. You also leave with the component/composable/store sorting instinct - the last architectural tool the capstone needs.

---

⬅️ [Module 2 · Reactivity](../module-02-reactivity/) · 🏠 [Course home](../README.md) · ➡️ [Start: Fetching data](./01-fetching-data.md)
