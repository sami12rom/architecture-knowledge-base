---
title: CSR (Clean Slice Redux)
type: pattern
tags: [backend, frontend, feature-sliced, vertical-slicing, modularity]
status: complete
created: 2026-03-22
updated: 2026-03-22
---

# CSR — Clean Slice Redux

> Organise code by business feature, not technical role. Everything related to a feature lives together in one slice.

Also referred to as **feature-sliced architecture** or **vertical slicing**.

---

## Problem it solves

The default way most projects start is horizontal layering: all controllers in one folder, all services in another, all repositories in a third. This feels clean at first because it mirrors the technical roles in the stack.

It stops working once the codebase grows.

When you need to change how orders are processed, you're touching `controllers/orders.ts`, `services/orders.ts`, `repositories/orders.ts`, and possibly `models/order.ts`. Four different folders, four different contexts to switch between — and this is for a single, self-contained feature. As the team grows and more features are added, the folders become noisy and navigating the codebase gets slow.

The horizontal model also makes ownership fuzzy. Which team owns the `services/` folder? All of them, which usually means none of them.

CSR flips the axis. Instead of organising by what something is technically, you organise by what it belongs to business-wise. Everything that exists to serve the orders feature lives in `orders/`. The folder becomes the unit of ownership.

---

## How it works

Each feature gets its own folder — a slice. Inside the slice, the standard technical layers still exist (controller, service, repository), but they're scoped to that feature.

```
src/
├── orders/
│   ├── orders.controller.ts
│   ├── orders.service.ts
│   ├── orders.repository.ts
│   ├── orders.module.ts
│   └── order.model.ts
├── payments/
│   ├── payments.controller.ts
│   ├── payments.service.ts
│   ├── payments.repository.ts
│   └── payment.model.ts
└── shared/
    ├── database/
    └── config/
```

### Key building blocks

| Building block | Role |
|---|---|
| **Slice** | One folder per business feature. Self-contained. |
| **Controller** | Entry point into the slice — handles HTTP, queue messages, CLI input, etc. |
| **Service** | Business logic for this feature. Calls repositories and external services. |
| **Repository** | Data access. Reads and writes to the database for this feature. |
| **Model** | The data shape. Can be a domain model, a DTO, or both depending on complexity. |
| **Shared** | Infrastructure and utilities that genuinely belong to no single feature. Keep this small. |

### The dependency rules

Cross-slice dependencies are allowed but should be explicit and minimal. If `payments` needs something from `orders`, that dependency should be a deliberate, visible import — not an implicit coupling through a shared service.

If you find yourself constantly importing between slices, that's a signal the slice boundaries are wrong, or that the shared code should move to `shared/`.

---

## Diagram

```
┌─────────────────────────────────────────────────────┐
│ src/                                                │
│                                                     │
│  ┌──────────────────┐    ┌──────────────────┐       │
│  │     orders/      │    │    payments/     │       │
│  │                  │    │                  │       │
│  │  controller      │    │  controller      │       │
│  │  service         │    │  service         │       │
│  │  repository      │    │  repository      │       │
│  │  model           │    │  model           │       │
│  └────────┬─────────┘    └────────┬─────────┘       │
│           │                       │                 │
│           └───────────┬───────────┘                 │
│                       │                             │
│              ┌────────▼────────┐                    │
│              │    shared/      │                    │
│              │  database       │                    │
│              │  config         │                    │
│              └─────────────────┘                    │
└─────────────────────────────────────────────────────┘
```

---

## When to use

- The codebase is growing and horizontal folders are becoming hard to navigate
- You want clear team ownership — one team owns one or more slices
- Most features are reasonably independent with limited cross-cutting logic
- You're working on a backend that's mostly CRUD or request/response with light business logic
- You want something that's easy for new engineers to understand without a long onboarding

---

## When not to use

- The domain logic is complex and needs protection from infrastructure — use [Hexagonal Architecture](../hexagonal-architecture/README.md) instead, or layer it inside each slice
- Features have so much shared logic that the slice boundaries feel arbitrary — this is usually a sign the domain model needs rethinking, not the folder structure
- You need fast, isolated unit tests for business logic — CSR services typically call repositories directly, which means tests often require a real database or heavy mocking

---

## Trade-offs

| Benefit | Cost |
|---|---|
| Easy to navigate — find anything related to a feature in one place | Shared logic is harder to place; `shared/` can become a dumping ground |
| Natural ownership boundaries for teams | Cross-slice dependencies can creep in if not actively managed |
| Low learning curve — most engineers understand it immediately | Business logic inside services becomes hard to test without infrastructure |
| Scales well for large teams with clear domain boundaries | Without further structure (e.g. ports), swapping infrastructure is painful |

---

## Code sketch

```typescript
// orders/order.model.ts
export interface Order {
  id: string;
  customerId: string;
  items: OrderItem[];
  status: 'pending' | 'confirmed' | 'shipped';
  total: number;
}

// orders/orders.repository.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Order } from './order.model';

@Injectable()
export class OrdersRepository {
  constructor(
    @InjectRepository(Order)
    private readonly repo: Repository<Order>,
  ) {}

  async findById(id: string): Promise<Order | null> {
    return this.repo.findOneBy({ id });
  }

  async save(order: Order): Promise<Order> {
    return this.repo.save(order);
  }
}

// orders/orders.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { OrdersRepository } from './orders.repository';
import { Order } from './order.model';

@Injectable()
export class OrdersService {
  constructor(private readonly orders: OrdersRepository) {}

  async confirm(orderId: string): Promise<Order> {
    const order = await this.orders.findById(orderId);
    if (!order) throw new NotFoundException('Order not found');
    if (order.status !== 'pending') throw new Error('Order already processed');

    order.status = 'confirmed';
    return this.orders.save(order);
  }
}

// orders/orders.controller.ts
import { Controller, Patch, Param } from '@nestjs/common';
import { OrdersService } from './orders.service';

@Controller('orders')
export class OrdersController {
  constructor(private readonly ordersService: OrdersService) {}

  @Patch(':id/confirm')
  confirm(@Param('id') id: string) {
    return this.ordersService.confirm(id);
  }
}
```

The slice is self-contained. The controller, service, and repository all live together, and nothing outside `orders/` needs to know how this works internally.

---

## Real-world examples

- **NestJS** promotes this structure out of the box with its module system — each module is effectively a slice
- **Ruby on Rails engines** apply the same idea at a larger scale — a self-contained Rails engine is a slice
- Most microservice architectures that haven't fully committed to Hexagonal land here naturally — one service per domain, horizontal layers inside

---

## Related

- [Hexagonal Architecture](../hexagonal-architecture/README.md) — adds explicit port/adapter boundaries inside each slice for better testability and infrastructure isolation
- [CSR vs Hexagonal](../../comparisons/csr-vs-hexagonal.md) — a direct comparison of both approaches and when to combine them
- [Coupling & Cohesion](../../concepts/coupling-and-cohesion.md) — CSR improves cohesion at the feature level; it doesn't automatically reduce coupling to infrastructure

---

## Further reading

- [Feature Sliced Design](https://feature-sliced.design/) — a formalised methodology for frontend that applies the same vertical slicing idea
- [Vertical Slice Architecture — Jimmy Bogard](https://www.jimmybogard.com/vertical-slice-architecture/) — a detailed write-up on applying this to .NET backends, principles transfer to any stack
