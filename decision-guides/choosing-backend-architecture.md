---
title: Choosing a Backend Architecture
type: decision-guide
tags: [backend, decision, architecture-selection]
status: complete
created: 2026-03-22
updated: 2026-03-22
---

# Choosing a Backend Architecture

Work through these questions in order. Stop when you have a clear answer.

---

## 1. How complex is your domain logic?

**Mostly CRUD — reading, writing, basic transformations?**
Use a simple layered structure or CSR (feature slices). Don't add complexity you don't need yet.

**Rich business rules — workflows, invariants, things that could go wrong in non-obvious ways?**
Keep reading.

---

## 2. How many things drive your use cases?

**One delivery mechanism — a REST API, a background worker, one entry point?**
CSR with a service/repository pattern is enough. Add explicit ports later if you feel the friction.

**Multiple entry points — REST and a CLI and a queue consumer all triggering the same logic?**
This is where Hexagonal pays off. The use case lives in the domain; each entry point is just a driving adapter. Without this structure, you end up duplicating logic or wiring the same thing in three places.

---

## 3. How stable is your infrastructure?

**You're not planning to change your database, messaging platform, or cloud setup any time soon?**
A repository pattern and some interface abstractions are enough. You don't need full Hexagonal.

**You're not sure — or you know a migration is coming?**
Hexagonal. Driven ports mean your domain doesn't care whether it's talking to Postgres, DynamoDB, or an in-memory fake. Infrastructure migrations become a matter of writing a new adapter, not excavating business logic.

---

## 4. What does testing look like for you?

**Integration tests are acceptable and run fast enough?**
Any layered approach works fine. Test through the HTTP layer and call it done.

**You need business logic tests that don't touch the database?**
Hexagonal. The domain is pure. In tests, you swap real adapters for in-memory fakes. Tests run in milliseconds and you can cover edge cases without standing anything up.

---

## 5. How big is the team?

**Solo or very small team?**
Start simple. CSR with service classes. You can introduce more structure when the complexity justifies it, and you'll know when that is.

**Multiple teams sharing a backend?**
Hexagonal with DDD Bounded Contexts. Team ownership maps to bounded contexts; ports define the interfaces between them. Without this, shared backends become coordination bottlenecks.

---

## Summary

| Situation | What to use |
|---|---|
| CRUD app, thin logic | Layered architecture or CSR |
| Growing domain, single entry point | CSR with repository pattern |
| Complex domain, multiple entry points | Hexagonal |
| Complex domain, uncertain infrastructure | Hexagonal |
| Multiple teams, complex domain | Hexagonal + DDD Bounded Contexts |
| Async, event-heavy workflows | Event-Driven Architecture |

---

## Common mistakes

**Adding Hexagonal to a CRUD app.** If your domain is just reading and writing records, you've added four layers of indirection with no business logic to protect. The pattern has a cost; make sure there's something worth paying it for.

**Staying with CSR too long on a complex domain.** Service classes work until they don't. When they become god objects and every test needs a database, the structural debt is already significant. Don't wait until it's painful to change.

**Treating any of this as permanent.** Starting with CSR and refactoring toward Hexagonal later is a completely valid path. It's much easier than a ground-up rewrite, and you'll have a clearer sense of where the boundaries actually need to be once the domain has had time to settle.

---

## Related

- [Hexagonal Architecture](../patterns/hexagonal-architecture/README.md)
- [CSR vs Hexagonal](../comparisons/csr-vs-hexagonal.md)
- [Coupling & Cohesion](../concepts/coupling-and-cohesion.md)
