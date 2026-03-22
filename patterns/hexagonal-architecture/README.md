---
title: Hexagonal Architecture
type: pattern
tags: [backend, layered, testability, ports-and-adapters]
status: complete
created: 2026-03-22
updated: 2026-03-22
---

# Hexagonal Architecture

> Keep business logic completely isolated from infrastructure by defining explicit ports and pluggable adapters.

Also called **Ports and Adapters**. Coined by Alistair Cockburn in 2005.

---

## Problem it solves

In most codebases, business logic gradually bleeds into controllers, database queries, and third-party clients. It usually starts small — a bit of logic in a controller here, a query that does more than it should there — and before long you have a codebase where:

- You can't test a business rule without spinning up a database or HTTP server
- Switching your ORM, messaging platform, or cloud provider means digging through core logic
- Every layer knows too much about every other layer

The pattern enforces a strict rule: the domain never reaches outward. All communication must flow inward, through interfaces the domain defines.

---

## How it works

Three layers:

### 1. Domain (the hexagon)
Pure business logic. No framework imports, no database calls, no HTTP. Just models, rules, and use cases. This is the part of the codebase that should outlive any infrastructure choice you make.

### 2. Ports (the interfaces)
Contracts defined by the domain — not by infrastructure. Two types:

| Port type | Direction | Example |
|---|---|---|
| **Driving (inbound)** | External → Domain | `OrderService` interface called by a REST controller |
| **Driven (outbound)** | Domain → External | `OrderRepository` interface implemented by a Postgres adapter |

The key distinction: ports are owned by the domain. Infrastructure must adapt to them, not the other way around.

### 3. Adapters
Concrete implementations that live outside the hexagon. A REST controller is a driving adapter. A Postgres repository is a driven adapter. The domain never knows or cares which adapter is in use.

```
┌────────────────────────────────────────────────┐
│                                                │
│   REST Controller ──► [Driving Port]           │
│                              │                 │
│                        ┌─────▼──────┐          │
│                        │   DOMAIN   │          │
│                        │  (use      │          │
│                        │   cases,   │          │
│                        │   models)  │          │
│                        └─────┬──────┘          │
│                              │                 │
│                       [Driven Port]            │
│                              │                 │
│            ┌─────────────────┼──────────────┐  │
│            │                 │              │  │
│     Postgres Adapter    SQS Adapter   Email │  │
│                                       Adapter  │
└────────────────────────────────────────────────┘
```

---

## When to use

- Your domain logic is complex enough to be worth protecting from infrastructure churn
- You need to test business rules without standing up a database
- Multiple things drive the same use case — a REST endpoint, a CLI command, a queue consumer
- The team is large enough that clear ownership boundaries actually matter
- You know (or suspect) that infrastructure decisions will change — DB migrations, messaging platforms, cloud moves

---

## When not to use

- It's a CRUD app with no real business logic. If your "domain" is just reading and writing records, there's nothing to protect and the extra layers cost more than they save.
- You're building a prototype or an internal tool that needs to move fast. The pattern adds structure; structure has upfront cost.
- The team hasn't used it before and there's no time to invest in the learning curve. Badly applied hexagonal architecture is worse than no architecture.

---

## Trade-offs

| Benefit | Cost |
|---|---|
| Business logic is testable without any infrastructure | More files, more interfaces, more indirection to navigate |
| Infrastructure can be swapped without touching the domain | Easy to leak dependencies inward if the team isn't disciplined |
| The same use case can be driven from multiple entry points | Defining ports upfront requires you to think before you code |
| Clear boundary between domain and infrastructure ownership | Boilerplate for operations that didn't need abstraction |

---

## Code sketch

```typescript
// --- DOMAIN ---

// Domain model
interface Order {
  id: string;
  items: OrderItem[];
  status: 'pending' | 'confirmed' | 'shipped';
}

// Driven port — the domain defines this, infrastructure implements it
interface OrderRepository {
  findById(id: string): Promise<Order | null>;
  save(order: Order): Promise<void>;
}

// Use case — business logic only, no infrastructure awareness
class ConfirmOrderUseCase {
  constructor(private readonly orders: OrderRepository) {}

  async execute(orderId: string): Promise<void> {
    const order = await this.orders.findById(orderId);
    if (!order) throw new Error('Order not found');
    if (order.status !== 'pending') throw new Error('Order already processed');

    order.status = 'confirmed';
    await this.orders.save(order);
  }
}

// --- ADAPTER (infrastructure) ---

class PostgresOrderRepository implements OrderRepository {
  async findById(id: string): Promise<Order | null> {
    return db.query('SELECT * FROM orders WHERE id = $1', [id]);
  }

  async save(order: Order): Promise<void> {
    await db.query('UPDATE orders SET status = $1 WHERE id = $2', [order.status, order.id]);
  }
}

// --- WIRING (composition root) ---

const orderRepo = new PostgresOrderRepository();
const confirmOrder = new ConfirmOrderUseCase(orderRepo);

// In tests, replace PostgresOrderRepository with an in-memory fake.
// No database. No mocking framework. Fast.
```

---

## Real-world examples

- **Netflix** — streaming business rules are isolated so they can test and swap delivery infrastructure without touching core logic
- **Spotify** — the same playlist and library logic runs across mobile, desktop, and embedded devices through different adapters
- Any NestJS backend using interface injection and the module/provider pattern is applying this — often without naming it

---

## Related

- [Clean Architecture](../clean-architecture/README.md) — a superset that adds more explicit layer rings (Robert C. Martin's version)
- [Layered Architecture](../layered-architecture/README.md) — the simpler predecessor; no port abstraction, unidirectional dependencies
- [Coupling & Cohesion](../../concepts/coupling-and-cohesion.md) — the underlying forces this pattern is designed to manage

---

## Further reading

- [Alistair Cockburn's original article (2005)](https://alistair.cockburn.us/hexagonal-architecture/) — still the clearest explanation, worth reading before anything else
- [Explicit Architecture — Herbert Graça](https://herbertograca.com/2017/11/16/explicit-architecture-01-ddd-hexagonal-onion-clean-cqrs-how-i-put-it-all-together/) — how Hexagonal fits with DDD, Onion, Clean, and CQRS
