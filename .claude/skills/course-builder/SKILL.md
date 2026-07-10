---
name: course-builder
description: Build and extend self-paced training courses as GitHub-browsable markdown. Use this whenever the user wants to learn a topic, or mentions a course, learning path, study plan, curriculum, or tutorial series - even if they don't say the word "course". Also use when asked to write or continue the next module of an existing course in this repo, or when a directory README describes itself as a learning path that needs content.
---

# Course Builder

Create self-paced training courses as markdown that reads beautifully on github.com - no build step, no site generator, just files and folders you can browse.

## Who this is for

The learner (Steve) has a non-linear brain and does not absorb big walls of text. This shapes everything:

- **Chunk ruthlessly.** No section longer than ~5 short paragraphs. If an explanation is getting long, it wants to be a diagram, a table, a code sample, or two smaller sections.
- **Show before tell.** Lead with a diagram, an annotated code snippet, or a concrete example - then explain it. Abstract prose first is the failure mode.
- **Support jumping around.** Every course gets a visual map with prerequisite arrows so the learner can see what depends on what and pick their own path. Never assume they read the previous page.
- **Depth on demand.** Put deep-dives, solutions, and quiz answers inside `<details>` blocks so the main flow stays light and the reader chooses when to go deeper.
- **Do, don't just read.** Every lesson ends with a hands-on exercise and a short checkpoint quiz. Learning happens in the doing.
- **No em-dashes.** Never use "—" anywhere in course content; it's a hard style preference. Use a plain hyphen with spaces, a comma, a colon, or split the sentence.

## Workflow: creating a new course

### 1. Quick interview (keep it short)

Ask only what you can't infer - usually just:
- What do they already know about the topic? (sets the starting level)
- What do they want to be able to *do* at the end? (sets the finish line)

Don't interrogate. Two questions, then move.

### 2. Research the syllabus

Before writing anything, make sure the course reflects the topic *as it is today*. If web search is available, check current versions, current best practices, and what the official docs/tutorials cover - a course teaching a deprecated API is worse than no course. Sketch 4–7 modules that take the learner from their starting level to their goal.

### 3. Build the course home

Create the course directory (inside the repo area the user indicates - e.g. `PersonalDevelopment/Dev/<Topic>/`) with a `README.md` containing:

- **One-line pitch** - what you'll be able to do after this course.
- **Course map** - a Mermaid flowchart of modules with prerequisite arrows. Modules that don't depend on each other should visibly branch, inviting non-linear exploration. (GitHub sandboxes Mermaid interactivity, so nodes can't be links - the map shows the shape, the module table right below it carries the links.)
- **Module table** - number, title (linked to the module), one-line "what you'll build/learn", status (✅ written / 📝 planned).
- **Progress tracker** - a GitHub task list (`- [ ]`) of modules; GitHub renders these as real checkboxes the learner ticks by editing the file.

### 4. Write Module 1 in full, stub the rest

Write the first module completely: a module `README.md` (intro + lesson list) and 2–4 lessons following [references/lesson-template.md](references/lesson-template.md) - read that file before writing your first lesson.

For every other module, create just its `README.md` stub: learning objectives, planned lesson titles, and a note like *"Not written yet - ask Claude to 'write module 3' when you get here."* Stubs keep the map honest and make continuing the course a one-line request.

### 5. Wire up navigation

- Every lesson gets a nav footer: `⬅️ Previous · 🗺️ Course map · Next ➡️` using relative links.
- Link the new course from the repo's root `README.md` index if one exists.
- Use **relative links everywhere** so navigation works on github.com, in clones, and in editors alike.

## Workflow: continuing an existing course

When asked to write the next module (or a specific one):

1. Read the course `README.md` (map, module table) and skim the most recent written module - match its voice, depth, and formatting exactly. A course that changes personality halfway through is jarring.
2. Write the module's lessons per the template.
3. Update the course home: flip the module's status to ✅, and make sure the map still reflects reality.

## Directory layout

```
<Topic>/
├── README.md                  # course home: pitch, map, module table, progress
├── module-01-<slug>/
│   ├── README.md              # module intro + lesson list
│   ├── 01-<lesson-slug>.md
│   └── 02-<lesson-slug>.md
├── module-02-<slug>/
│   └── README.md              # stub until written
└── ...
```

Number modules and lessons with zero-padded prefixes (`module-01`, `01-`) so GitHub's file listing sorts them in course order - the directory listing itself becomes a table of contents.

## GitHub-flavored markdown toolkit

These all render natively on github.com - use them instead of plain prose wherever they carry the idea better:

| Instead of... | Use |
|---|---|
| A paragraph describing structure or flow | Mermaid diagram (` ```mermaid `) |
| "Note that..." / "Be careful..." prose | Alerts: `> [!NOTE]`, `> [!TIP]`, `> [!WARNING]`, `> [!IMPORTANT]` |
| Inline solutions and quiz answers | `<details><summary>Show answer</summary>` blocks |
| A prose comparison of options | A table |
| "First do X, then Y..." | A numbered list or task list |

> [!IMPORTANT]
> Mermaid node labels containing parentheses or special characters need quoting: `A["setup() function"]`. Preview mentally for syntax errors - a broken diagram on the course map breaks the whole navigation experience.

## Quality bar

Before finishing, walk the course as the learner would: start at the course home, click into module 1, follow the nav footers. Every link must resolve, every diagram must render, and no page should confront the reader with an unbroken screen of text.
