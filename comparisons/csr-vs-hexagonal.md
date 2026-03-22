---
title: CSR (Clean Slice Redux) vs Hexagonal Architecture
type: comparison
tags: [backend, frontend, layered, testability, ports-and-adapters]
status: complete
created: 2026-03-22
updated: 2026-03-22
---

# CSR vs Hexagonal Architecture

> One organises code by feature. The other organises code by distance from the domain. They solve different problems, and you can use both at once.

---

## The problem each one is solving

As a backend grows, two things tend to break down.

The first is **discoverability**. When every controller lives in one folder and every service in another, finding everything related to "orders" means jumping around the codebase. This gets painful fast.

The second is **infrastructure leakage**. Business logic creeps into SQL queries, HTTP handlers, and third-party client calls. Eventually you can't test a business rule without spinning up a database, and changing your ORM becomes a multi-day project.

CSR fixes the first problem. Hexagonal fixes the second. That's why they're often used together.

---

## What is CSR (Clean Slice Redux)?

CSR — also called feature-sliced architecture or vertical slicing — organises code by business feature, not technical role.

```
src/
├── orders/
│   ├── orders.controller.ts
│   ├── orders.service.ts
│   ├── orders.repository.ts
│   └── orders.model.ts
├── payments/
│   ├── payments.controller.ts
│   └── ...
```

Each slice owns everything it needs end-to-end. The rule is simple: if it belongs to orders, it lives in orders. Cross-slice dependencies are allowed but should be explicit and minimal.

The payoff is discoverability. A new engineer can find everything related to a feature in one place.

---

## What is Hexagonal Architecture?

Hexagonal organises code by its relationship to the domain. The question isn't "is this orders or payments?" — it's "is this business logic, or is this infrastructure?"

```
src/
├── domain/
│   ├── order.model.ts
│   ├── confirm-order.use-case.ts
│   └── ports/
│       ├── order-repository.port.ts
│       └── payment-gateway.port.ts
├── adapters/
│   ├── http/
│   │   └── orders.controller.ts
│   ├── persistence/
│   │   └── postgres-order-repository.ts
│   └── external/
│       └── stripe-payment-gateway.ts
```

The domain defines interfaces (ports) for anything it needs from the outside world. Infrastructure implements them. The domain never knows which implementation is in use, which means you can swap Postgres for DynamoDB — or swap real infrastructure for an in-memory fake in tests — without touching business logic.

---

## Side-by-side

| | CSR | Hexagonal |
|---|---|---|
| Organises by | Business feature | Distance from domain |
| Infrastructure isolation | Low — services often call the DB directly | High — only adapters touch infrastructure |
| Testability of business logic | Moderate — usually requires mocking | High — domain is pure, no mocking needed |
| Adding a new feature | Easy — clone a slice | More setup — define ports first |
| Swapping infrastructure | Hard — changes spread across slices | Easy — swap one adapter |
| Works with DDD | Partially | Naturally |
| Learning curve | Low | Medium |

---

## They're not mutually exclusive

This is the part people miss. You can apply CSR at the directory level and Hexagonal within each slice.

```
src/
├── orders/
│   ├── domain/
│   │   ├── order.model.ts
│   │   └── confirm-order.use-case.ts
│   ├── ports/
│   │   └── order-repository.port.ts
│   └── adapters/
│       ├── orders.controller.ts
│       └── postgres-order-repository.ts
```

You get feature co-location (everything orders-related is in `orders/`) and infrastructure isolation (the domain is still clean). This is the structure I'd reach for in a codebase that's growing beyond a single team.

---

## Which to choose

**Start with CSR** if you're moving fast, the domain is simple, or the team is small. The structure is intuitive and the overhead is low.

**Introduce Hexagonal** when you find yourself mocking database calls in every test, or when changing a data store becomes more work than it should be. You'll feel the need for it before you can articulate why — and that's usually the right time to add it.

**Combine them** when the codebase is large enough that different features have different complexity levels. Some slices will warrant full hexagonal layering. Others won't. That's fine.

The mistake is treating them as either/or. They weren't designed to compete.

---

## Related

- [Hexagonal Architecture](../patterns/hexagonal-architecture/README.md)
- [Coupling & Cohesion](../concepts/coupling-and-cohesion.md)
- [Choosing a Backend Architecture](../decision-guides/choosing-backend-architecture.md)
