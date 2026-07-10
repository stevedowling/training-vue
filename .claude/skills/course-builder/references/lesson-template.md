# Lesson Template

Every lesson follows this shape. The order matters: hook → picture → chunks → practice → check. It front-loads motivation and imagery because the learner disengages from abstract prose, and it ends with doing because that's where the learning sticks.

Target length: a lesson should be readable in 10–15 minutes. If it's longer, split it into two lessons.

````markdown
# <Lesson number> · <Lesson title>

> **You'll learn:** <one sentence - the capability, not the topic. "Make a component react to user input", not "Event handling".>

## Why this matters

<2–3 sentences max. A concrete situation where you'd need this. If you can't explain why it matters, question whether the lesson belongs in the course.>

## The big picture

<A Mermaid diagram, annotated code sample, or before/after comparison that shows the whole idea at a glance. The reader should get the gist from this section alone - the rest of the lesson is elaboration.>

## <Concept chunk 1>

<Short. A few sentences plus a code example or table. One idea per heading.>

## <Concept chunk 2>

<Same. Use `> [!TIP]` / `> [!WARNING]` alerts for gotchas instead of burying them in prose.>

<details>
<summary>🔍 Deep dive: <optional advanced tangent></summary>

<Anything interesting-but-not-essential goes here, keeping the main flow light.>

</details>

## 🛠️ Try it

<A hands-on exercise using what was just covered. Concrete deliverable: "Build X that does Y." State where to put the work (e.g. an `exercises/` folder or a scratch project). Progressive hints beat a monolithic solution:>

<details>
<summary>💡 Hint 1</summary>
<nudge, not answer>
</details>

<details>
<summary>✅ Solution</summary>

```<lang>
<working solution with brief comments>
```

</details>

## ✋ Checkpoint

<2–4 quick questions. Prediction questions ("what does this print?") beat recall questions ("what is X?").>

1. <question>
2. <question>

<details>
<summary>Answers</summary>

1. <answer with one-line explanation>
2. <answer with one-line explanation>

</details>

## 📚 Further reading

- [<Official docs page for this topic>](url) - <why it's worth opening>
- <1–3 more curated links; quality over quantity>

---

⬅️ [Previous: <title>](<relative-path>) · 🗺️ [Course map](../README.md) · ➡️ [Next: <title>](<relative-path>)
````

## Notes on filling the template

- **"You'll learn" is a promise** - end-of-lesson exercise must deliver on it exactly.
- **Checkpoint answers explain, not just state.** A wrong guess followed by a one-line "because..." is a teaching moment.
- **Further reading is curated, not exhaustive.** Each link needs a reason to click it.
- **First and last lessons of a module** may drop sections that don't fit (an intro lesson might not need an exercise; a capstone lesson might be *all* exercise). The template serves the lesson, not the other way around.
