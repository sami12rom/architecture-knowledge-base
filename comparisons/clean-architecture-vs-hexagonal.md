---
title: Clean Architecture vs Hexagonal Architecture
type: comparison
tags: [backend, layered, testability, ports-and-adapters, domain, use-cases]
status: complete
created: 2026-03-22
updated: 2026-03-22
---

# Clean Architecture vs Hexagonal Architecture

> They share the same core idea — dependencies point inward, infrastructure is a detail — but they differ in how much structure they prescribe inside that boundary.

This is one of the most common sources of confusion in backend architecture. Engineers often use the names interchangeably, or assume one is a replacement for the other. Neither is quite right.

---

## What they share

Both patterns enforce the same fundamental rule: **business logic must not depend on infrastructure**.

In both:
- The domain (entities, business rules, use cases) sits at the centre
- Infrastructure (databases, HTTP, messaging) sits at the edge
- The domain communicates with infrastructure through interfaces, never directly
- Swapping infrastructure — changing your database, your messaging platform, your delivery mechanism — should not require touching business logic

If you understand one deeply, the other will feel familiar.

---

## Where they differ

The difference is in what they prescribe *inside* the boundary.

**Hexagonal Architecture** focuses on the boundary itself. It says: define ports (interfaces) that your domain depends on, and write adapters that implement them. It doesn't tell you much about how to organise the domain internally. There's no prescribed separation between entities and use cases. The domain is just "the inside of the hexagon".

**Clean Architecture** goes further. It prescribes explicit rings inside the boundary:

1. **Entities** — business objects with rules baked in
2. **Use Cases** — orchestration logic that calls entities and coordinates ports
3. **Interface Adapters** — translation between use cases and infrastructure
4. **Frameworks & Drivers** — the actual infrastructure, treated as a plug-in

The Dependency Rule applies not just at the outer boundary but between every ring. A use case can depend on an entity, but an entity cannot depend on a use case.

---

## Side-by-side

| | Hexagonal | Clean Architecture |
|---|---|---|
| Core concept | Ports and adapters | Concentric rings with strict dependency rules |
| Internal domain structure | Not prescribed | Entities and Use Cases are explicit, separate rings |
| Business rule location | Anywhere inside the hexagon | Specifically on Entities |
| Use case orchestration | Typically in a service or handler class | Explicitly separated into a Use Case ring |
| Adapter concept | Explicit (driving and driven adapters) | Covered by the Interface Adapters ring |
| How strict are the rules | One rule: domain doesn't depend on infrastructure | One rule per ring boundary |
| Learning curve | Lower | Higher |
| Verbosity | Less boilerplate | More boilerplate |

---

## The key practical difference: where business rules live

This is the most important distinction and the one that gets glossed over most often.

In Hexagonal, there's no rule about whether business logic lives in a service class or on an entity. Many Hexagonal implementations put all logic in service/use case classes and treat entities as plain data objects. This works, but it means the entities are just DTOs — they don't enforce their own rules.

Clean Architecture is explicit: **business rules belong on entities**. The entity is responsible for enforcing its invariants. The use case just orchestrates — it loads entities, calls methods on them, and persists the result.

```typescript
// Hexagonal style — logic tends to live in the use case/service
class ConfirmOrderService {
  async confirm(orderId: string): Promise<void> {
    const order = await this.orders.findById(orderId);
    if (order.status !== 'pending') throw new Error('Already processed'); // rule here
    order.status = 'confirmed';
    await this.orders.save(order);
  }
}

// Clean Architecture style — rule belongs to the entity
class Order {
  confirm(): void {
    if (this.status !== 'pending') throw new Error('Already processed'); // rule here
    this.status = 'confirmed';
  }
}

class ConfirmOrderUseCase {
  async execute(orderId: string): Promise<void> {
    const order = await this.orders.findById(orderId);
    order.confirm(); // use case just orchestrates
    await this.orders.save(order);
  }
}
```

The second version is more testable at the entity level and keeps use cases thin. The first version is simpler but puts all the logic in one place, which gets crowded as complexity grows.

---

## Which to choose

**Choose Hexagonal** when:
- You need infrastructure isolation and good testability, but the domain isn't complex enough to warrant separating entities from use cases
- The team is new to this style of architecture and you want a lower barrier to entry
- You want to adopt the pattern incrementally — Hexagonal is easier to introduce into an existing codebase

**Choose Clean Architecture** when:
- Your domain has rich business rules that warrant being baked into entities
- You want explicit, enforced boundaries between orchestration logic and business rules
- The codebase will be long-lived and maintained by multiple teams — the additional structure pays off over time
- You're already practising DDD, where the entity/use case distinction maps directly to Aggregates and Application Services

**In practice, the two often converge.** A well-implemented Hexagonal codebase with a complex domain will naturally start separating entity behaviour from use case orchestration — which is Clean Architecture. And a Clean Architecture implementation is, by definition, a Hexagonal Architecture too. The rings are hexagonal; the ports and adapters are what connect the Interface Adapters ring to Frameworks & Drivers.

---

## Visualising the overlap

```
Hexagonal Architecture:

          [Driving Adapters]
                 │
          [Driving Ports]
                 │
         ┌───────────────┐
         │               │
         │    DOMAIN      │
         │  (use cases,  │
         │   entities,   │
         │   anything)   │
         │               │
         └───────────────┘
                 │
          [Driven Ports]
                 │
          [Driven Adapters]


Clean Architecture adds structure inside the domain:

         ┌───────────────────────────┐
         │   Interface Adapters      │  ← Driving + Driven Adapters
         │   ┌───────────────────┐   │
         │   │   Use Cases       │   │  ← Application ring
         │   │  ┌─────────────┐  │   │
         │   │  │  Entities   │  │   │  ← Domain ring
         │   │  └─────────────┘  │   │
         │   └───────────────────┘   │
         └───────────────────────────┘
```

---

## Related

- [Hexagonal Architecture](../patterns/hexagonal-architecture/README.md)
- [Clean Architecture](../patterns/clean-architecture/README.md)
- [Coupling & Cohesion](../concepts/coupling-and-cohesion.md)
- [Choosing a Backend Architecture](../decision-guides/choosing-backend-architecture.md)
