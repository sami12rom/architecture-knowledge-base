---
title: Clean Architecture
type: pattern
tags: [backend, layered, testability, domain, use-cases]
status: complete
created: 2026-03-22
updated: 2026-03-22
---

# Clean Architecture

> Organise code into concentric rings where dependencies only point inward — business rules at the centre, infrastructure at the edge.

Introduced by Robert C. Martin (Uncle Bob) in 2012. Clean Architecture is a synthesis of several earlier ideas — Hexagonal Architecture, Onion Architecture, and others — unified under one consistent model.

---

## Problem it solves

Most architectural failures come down to the same root cause: business logic that knows too much about the outside world. It knows which database you're using, which HTTP framework, which message broker. When any of those change, the business logic changes with them — and that's exactly backwards.

Clean Architecture makes this impossibility a rule. The inner layers cannot reference the outer layers. Ever. The database doesn't drive the design; the use cases do. Infrastructure is a detail, and details live at the edge.

The result is a codebase where you can swap your database, your delivery mechanism, or your UI framework without touching the rules that make your software actually work.

---

## How it works

The architecture is a set of concentric rings. Each ring can only depend on the rings inside it, never outside.

```
         ┌──────────────────────────────────┐
         │         Frameworks & Drivers      │  ← Web, DB, UI, external APIs
         │   ┌──────────────────────────┐   │
         │   │      Interface Adapters   │   │  ← Controllers, Presenters, Gateways
         │   │   ┌──────────────────┐   │   │
         │   │   │   Application    │   │   │  ← Use Cases
         │   │   │  ┌───────────┐   │   │   │
         │   │   │  │  Domain   │   │   │   │  ← Entities, Business Rules
         │   │   │  └───────────┘   │   │   │
         │   │   └──────────────────┘   │   │
         │   └──────────────────────────┘   │
         └──────────────────────────────────┘
                  dependencies flow inward →
```

### The four rings

**Entities (innermost)**
Core business objects and the rules that apply to them. These have no dependencies. An `Order` entity that enforces "you can't confirm an already-shipped order" — that rule belongs here. It should read identically whether this is a web app, a CLI, or a batch job.

**Use Cases (Application)**
Orchestrates entities to carry out a specific business operation. A use case like `ConfirmOrder` knows which entities to load, what rules to enforce, and what needs to happen as a result. It does not know about HTTP, databases, or any specific framework. It depends on interfaces (ports) for anything it needs from the outside world.

**Interface Adapters**
Translates between the outside world and the use cases. Controllers receive HTTP requests and call use cases. Presenters take the output of use cases and format it for a response. Repository implementations translate between your database and the domain model. This ring knows about both the use cases and the infrastructure — it's the translation layer between them.

**Frameworks & Drivers (outermost)**
The actual infrastructure: Express, NestJS, TypeORM, Postgres, Redis, SQS. These are treated as interchangeable details. You configure them here and wire them to the adapters. Nothing in the inner rings knows they exist.

### Key building blocks

| Building block | Ring | Role |
|---|---|---|
| Entity | Domain | Business object + the rules baked into it |
| Use Case | Application | Orchestrates entities for one business operation |
| Port (interface) | Application | Contract the use case defines for anything it needs |
| Controller | Interface Adapters | Receives input, calls use case |
| Presenter / DTO | Interface Adapters | Formats use case output for delivery |
| Repository implementation | Interface Adapters | Translates between database and domain |
| Framework config | Frameworks & Drivers | Wires everything together |

---

## When to use

- Complex domain with business rules that need to be tested independently of infrastructure
- Long-lived systems where infrastructure choices will change over time
- Multiple delivery mechanisms sharing the same core logic (REST API, CLI, background worker)
- Teams where domain ownership and infrastructure ownership are separate concerns
- When you want to enforce strict architectural rules that survive team turnover

---

## When not to use

- CRUD applications with minimal business logic. The four rings add significant structure — it only pays off when there's meaningful domain logic to protect.
- Small teams or early-stage projects where the overhead slows you down more than the structure helps
- When the team isn't bought in. Clean Architecture is opinionated and requires consistent discipline. Partial adoption often produces worse results than a simpler, consistent approach.

---

## Trade-offs

| Benefit | Cost |
|---|---|
| Business rules are completely isolated and independently testable | A lot of boilerplate — interfaces, DTOs, adapters for operations that may not need them |
| Infrastructure is genuinely swappable | The strict ring boundaries require constant discipline to maintain |
| Use cases are self-documenting — reading one tells you exactly what a feature does | The learning curve is real; misapplied Clean Architecture is worse than none |
| The architecture can outlast any infrastructure decision | Over-engineering risk on simpler domains |

---

## Code sketch

```typescript
// --- DOMAIN (entities) ---

export class Order {
  constructor(
    public readonly id: string,
    public readonly customerId: string,
    private status: 'pending' | 'confirmed' | 'shipped',
  ) {}

  confirm(): void {
    if (this.status !== 'pending') {
      throw new Error(`Cannot confirm an order with status: ${this.status}`);
    }
    this.status = 'confirmed';
  }

  getStatus() {
    return this.status;
  }
}

// --- APPLICATION (use case + port) ---

// Port — the use case defines this interface; infrastructure implements it
export interface OrderRepository {
  findById(id: string): Promise<Order | null>;
  save(order: Order): Promise<void>;
}

export interface ConfirmOrderOutput {
  orderId: string;
  newStatus: string;
}

export class ConfirmOrderUseCase {
  constructor(private readonly orders: OrderRepository) {}

  async execute(orderId: string): Promise<ConfirmOrderOutput> {
    const order = await this.orders.findById(orderId);
    if (!order) throw new Error('Order not found');

    order.confirm(); // business rule lives on the entity, not here

    await this.orders.save(order);

    return { orderId: order.id, newStatus: order.getStatus() };
  }
}

// --- INTERFACE ADAPTERS ---

// Controller — translates HTTP into a use case call
export class OrdersController {
  constructor(private readonly confirmOrder: ConfirmOrderUseCase) {}

  async handlePatch(req: Request, res: Response): Promise<void> {
    const result = await this.confirmOrder.execute(req.params.id);
    res.json(result);
  }
}

// Repository implementation — translates between DB and domain
export class PostgresOrderRepository implements OrderRepository {
  async findById(id: string): Promise<Order | null> {
    const row = await db.query('SELECT * FROM orders WHERE id = $1', [id]);
    if (!row) return null;
    return new Order(row.id, row.customer_id, row.status);
  }

  async save(order: Order): Promise<void> {
    await db.query('UPDATE orders SET status = $1 WHERE id = $2', [
      order.getStatus(),
      order.id,
    ]);
  }
}

// --- FRAMEWORKS & DRIVERS (wiring) ---

const orderRepo = new PostgresOrderRepository();
const confirmOrder = new ConfirmOrderUseCase(orderRepo);
const controller = new OrdersController(confirmOrder);
```

Notice the business rule (`Cannot confirm an order that isn't pending`) lives on the entity — not the use case, not the controller. The use case just orchestrates. This is the distinction Clean Architecture enforces that CSR and basic Hexagonal often don't.

---

## Real-world examples

- **Large fintech backends** commonly apply this pattern — financial rules need to be testable in isolation and the infrastructure (payment rails, databases, regulatory reporting) changes often
- **Enterprise Java/Spring applications** frequently mirror this structure with the package hierarchy `domain`, `application`, `adapters`, `infrastructure`
- The pattern is also the basis for most DDD implementations — Aggregates are entities, Application Services are use cases

---

## Related

- [Hexagonal Architecture](../hexagonal-architecture/README.md) — Clean Architecture is a superset; Hexagonal focuses on the port/adapter boundary without prescribing the internal ring structure
- [CSR — Clean Slice Redux](../csr/README.md) — feature organisation that can be combined with Clean Architecture within each slice
- [Clean Architecture vs Hexagonal](../../comparisons/clean-architecture-vs-hexagonal.md) — a direct comparison of where they overlap and where they differ
- [Coupling & Cohesion](../../concepts/coupling-and-cohesion.md) — the dependency rule is a direct enforcement of low coupling between rings

---

## Further reading

- [Clean Architecture — Robert C. Martin (2017)](https://www.informit.com/store/clean-architecture-a-craftsmans-guide-to-software-structure-9780134494166) — the book; worth reading in full, chapters 22–34 are the most directly applicable
- [The Clean Architecture blog post (2012)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) — the original article that introduced the model, shorter than the book
- [Explicit Architecture — Herbert Graça](https://herbertograca.com/2017/11/16/explicit-architecture-01-ddd-hexagonal-onion-clean-cqrs-how-i-put-it-all-together/) — how Clean Architecture, Hexagonal, Onion, and DDD fit together
