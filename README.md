# Architecture Knowledge Base

A reference for software architecture patterns, trade-offs, and decisions. Written to be reused — for articles, talks, internal discussions, or just settling an argument in a PR review.

---

## How to navigate

| Directory | What's inside |
|---|---|
| [`patterns/`](patterns/) | Deep dives into individual architecture patterns |
| [`comparisons/`](comparisons/) | Side-by-side breakdowns of two or more approaches |
| [`concepts/`](concepts/) | Foundational ideas that underpin the patterns |
| [`decision-guides/`](decision-guides/) | Practical guides for choosing between approaches |
| [`presentations/`](presentations/) | Outlines and speaker notes for talks |

---

## Patterns

| Pattern | Summary | Tags |
|---|---|---|
| [Hexagonal Architecture](patterns/hexagonal-architecture/README.md) | Keep business logic away from infrastructure using ports and adapters | `backend` `layered` `testability` |
| [CSR — Clean Slice Redux](patterns/csr/README.md) | Organise code by business feature, not technical role | `backend` `frontend` `feature-sliced` `modularity` |
| [Clean Architecture](patterns/clean-architecture/README.md) | Concentric rings where dependencies only point inward — business rules at the centre, infrastructure at the edge | `backend` `layered` `testability` `domain` |

---

## Comparisons

| Comparison | Summary |
|---|---|
| [CSR vs Hexagonal Architecture](comparisons/csr-vs-hexagonal.md) | Organising by feature slice vs organising by layer — when each makes sense |
| [Clean Architecture vs Hexagonal](comparisons/clean-architecture-vs-hexagonal.md) | Same core idea, different levels of prescription inside the boundary |

---

## Concepts

| Concept | Summary |
|---|---|
| [Coupling & Cohesion](concepts/coupling-and-cohesion.md) | The two forces every architecture is trying to balance |

---

## Decision Guides

| Guide | Summary |
|---|---|
| [Choosing a Backend Architecture](decision-guides/choosing-backend-architecture.md) | A step-by-step guide for picking the right structure |

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to add a pattern, comparison, or concept.

Use [GitHub Issues](../../issues) to track ideas or flag gaps. Label new ideas as `draft`.
