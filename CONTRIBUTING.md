# Contributing

## Adding a pattern

1. Copy `patterns/_template.md` into a new folder: `patterns/your-pattern-name/README.md`
2. Fill in every section. Use `_TODO_` as a placeholder if you're not done yet, but don't leave sections empty — a partial entry is fine, a blank one isn't useful.
3. Diagrams go in `patterns/your-pattern-name/diagrams/`. SVG preferred, PNG is fine.
4. Add a row in the patterns table in the root `README.md`.
5. Open a PR with the label `pattern`.

## Adding a comparison

1. Create `comparisons/pattern-a-vs-pattern-b.md`.
2. The structure should be: shared context → how each approach handles it → trade-off table → a concrete recommendation. Don't write a comparison that ends with "it depends" and nothing else.
3. Add a row to the comparisons table in `README.md`.

## Adding a concept

1. Create `concepts/concept-name.md`.
2. Define it, give a real example, and link to the patterns that depend on it.
3. Add a row to the concepts table in `README.md`.

## Adding a decision guide

1. Create `decision-guides/your-guide.md`.
2. Use a question-by-question format. Every answer should lead somewhere — a pattern, a concept, or a clear recommendation. No dead ends.

## Adding a presentation

1. Create `presentations/your-talk-title/outline.md` with the slide structure and speaker notes.
2. If there's a live deck, link to it at the top (Google Slides, Keynote share link, etc.).
3. Don't commit the binary file — just the link. `.pptx` and `.key` files don't belong in git.

---

## Frontmatter

Every doc should start with this:

```yaml
---
title: Hexagonal Architecture
type: pattern          # pattern | comparison | concept | decision-guide
tags: [backend, layered, testability]
status: complete       # draft | in-progress | complete
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

This makes it possible to build tooling on top later (site generation, search, filtering by tag).

---

## Writing style

Write for a mid-level engineer who has solid fundamentals but hasn't necessarily seen this pattern before. Don't write for a junior who needs hand-holding, and don't write for a researcher who wants academic rigour. Somewhere in the middle.

Be direct about trade-offs. If a pattern has a real downside, say so. If one approach is usually better than another, say that too. Vague hedging ("it depends on your context") is only useful when followed by specifics about what it depends on.

One doc, one thing. A pattern doc is not the place to explain cohesion from scratch — that's what the concepts section is for. Link instead.
