# 1 · Planning the app - components, pages, state, data

> **You'll learn:** how to plan a Vue app on paper before writing code - and leave with the blueprint you'll build in lesson 2 and deploy in lesson 3.

## Why this matters

Every skill in this course was practiced on a scaffold someone else shaped. The capstone removes the scaffold - and the difference between a weekend of satisfying building and a weekend of thrashing is the hour of planning that happens first. Professionals sketch the component tree, the routes, and the state *before* the first file; today that's the entire deliverable.

## The big picture

The app: **Snackmart** - the sandbox's greatest hits, rebuilt as one coherent product. (Prefer your own idea? The planning method below works for anything - swap the subject, keep every step. A movie browser, a recipe box, and a hiking-trail log have all graduated from this course. The requirements in the table are the bar either way.)

| Requirement | Module it proves |
|---|---|
| Products fetched from a real API (dummyjson), with honest loading/error/empty states | 7 |
| Catalog page + product detail pages with URLs (`/products/:id`), plus search, plus a 404 | 5, 7 |
| A cart that works from any page (header badge, add from catalog or detail) and survives reload | 6, 2 |
| A checkout form - validated, polite, with at least one custom v-model input | 4 |
| Composed from proper components: cards, rows, badge, rating - props down, events up | 3 |
| Deployed at a public URL | 8 |

## Step 1: pages and routes

Start where the user starts - what screens exist and what are their addresses? Write the route table first; it's the app's skeleton:

| Path | View | Job |
|---|---|---|
| `/` | HomeView | greeting, featured products (a taste of the API) |
| `/catalog` | CatalogView | the full grid, category filters, search box |
| `/products/:id` | ProductView | one product: image, blurb, price, rating, add-to-cart |
| `/cart` | CartView | contents, quantities editable, total, → checkout |
| `/checkout` | CheckoutView | the validated form; success clears the cart |
| `/:pathMatch(.*)*` | NotFoundView | charm |

## Step 2: the component tree

For each view, sketch what it's *made of* - boxes in boxes, nothing fancier than indentation:

```text
App
├── AppHeader
│   ├── nav (RouterLinks)
│   └── CartBadge          ← reads cart store
├── RouterView
│   ├── CatalogView
│   │   ├── SearchBar      ← v-model, debounced
│   │   └── ProductCard ×n ← props: product · emits: add
│   ├── ProductView
│   │   ├── StarRating     ← the Module 4 custom input, reused
│   │   └── QtyStepper
│   ├── CartView
│   │   └── CartRow ×n     ← props: item · emits: setQty, remove
│   └── CheckoutView
│       └── (form fields + QtyStepper... whatever it needs)
└── AppFooter
```

The test for each box: could you write its props-and-emits contract in one line? (`ProductCard: takes product, announces add` - yes, that's a component. "Handles the whole catalog page including fetching and filtering and..." - that's a view doing its proper job, or a box that needs splitting.)

## Step 3: state - the sorting you already know

Walk every piece of data through Module 6's flowchart and Module 7's instinct. For Snackmart:

| Data | Verdict | Why |
|---|---|---|
| cart contents | **store** (`cart`) | distant consumers, must survive routes + reload |
| products from the API | composable per view (`useFetch`) | each page fetches what it needs; promote to a cache store only if refetching annoys you later |
| search text | local to CatalogView | one consumer... but it feeds a getter URL - the lesson 7-3 wiring |
| checkout form fields + touched flags | local to CheckoutView | dies with the page, and should |
| theme (if you keep it) | store (`settings`) | app-wide, persistent |

Plus the API plan, which for dummyjson is three lines worth writing down: list = `/products?limit=12`, search = `/products/search?q=`, detail = `/products/:id` - and the detail one means ProductView *fetches its own product* by route param (fresh pattern! params + useFetch, composing modules 5 and 7 for the first time).

> [!TIP]
> Notice what planning *didn't* include: no CSS decisions, no file-by-file inventory, no perfection. A plan is done when every requirement has a home and no box scares you. One page, one hour, maximum.

<details>
<summary>🔍 Deep dive: how professionals actually hold plans loosely</summary>

Real teams do exactly this exercise (often on a whiteboard, often called a "spike" when code is sketched too) - and then expect the plan to bend. The component tree will be wrong in one place: you'll discover CartRow and ProductCard want to share something, or the checkout wants a step you didn't draw. That's not planning failure; that's planning *working* - the point was never prophecy, it was making the big decisions (routes, state homes, component contracts) cheap to change by making them explicit. When the build diverges from PLAN.md, update the plan in one line and keep moving. A stale plan that was honest for an hour beats no plan every time. What planning is *for* is preventing the expensive class of mistake: state in the wrong home, spaghetti props, pages that can't be linked to - the ones that cost a rewrite instead of a rename.

</details>

## 🛠️ Try it - write PLAN.md

The exercise *is* the lesson: create `PLAN.md` in a fresh project folder (lesson 2 scaffolds the code; the plan comes first, pointedly):

1. **Decide**: Snackmart, or your own subject clearing the same six requirements. Write the one-line pitch at the top.
2. **Routes**: your table, path → view → job. Six-ish rows.
3. **Tree**: the indented sketch, with a one-line props/emits contract next to every reusable component.
4. **State**: your data table with a verdict + one-word reason per row. Every requirement from the big table must appear somewhere in steps 2-4 - check them off literally.
5. **API**: the two or three endpoints you'll hit, with their response shapes noted (browse them in a tab right now - `https://dummyjson.com/products/1` - and paste a trimmed sample into the plan).
6. Read it once as your harshest reviewer: any box you couldn't start building tomorrow morning? Fix the plan, not the mood.

<details>
<summary>✅ What a done plan looks like</summary>

One page. A pitch line, three tables (routes, state, endpoints), one indented tree with contracts, and six checked requirement boxes. If yours is longer, it's probably doing lesson 2's job early - trim it. If it's shorter, find the requirement without a home. There is deliberately no full sample to copy here: the plan being *yours* is the assessment.

</details>

## ✋ Checkpoint

1. Why does the route table come before the component tree, and not after?
2. Your plan has `products` in a Pinia store "because the catalog and the detail page both show products". Argue the per-view-useFetch side - then say what future pain would justify the store after all.
3. A box in your tree reads "CatalogView - fetches, filters, renders grid, handles cart adds, syncs search to URL". Healthy or not, and by what test?
4. What are the two artifacts lesson 2 expects to receive from you?

<details>
<summary>Answers</summary>

1. Routes are the user-visible skeleton and the hardest thing to change late (links, bookmarks, deploy config all depend on them) - the tree hangs *off* the pages, not vice versa.
2. The two pages want *different* data (a list vs one full record); dummyjson serves each in one cheap call; no staleness to manage. The store earns its place when you're refetching the same list on every visit and it feels slow - a cache is a solution to a *measured* problem.
3. Healthy - it's a *view*, and orchestrating a page is the view's one-line job. It would be unhealthy as a reusable component (fails the one-line contract test).
4. PLAN.md, and a decision - Snackmart or your own subject - you're committed enough to build tomorrow.

</details>

## 📚 Further reading

- [Thinking in React](https://react.dev/learn/thinking-in-react) - yes, React: the single best written version of "sketch the tree, find the state homes", and it maps 1:1 onto what you just did
- [dummyjson docs](https://dummyjson.com/docs/products) - your API's full menu; skim for anything that improves your plan

---

⬅️ [Module home](./README.md) · 🏠 [Course home](../README.md) · ➡️ [Next: The build](./02-the-build.md)
