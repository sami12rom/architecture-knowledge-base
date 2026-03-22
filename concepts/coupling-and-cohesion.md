---
title: Coupling & Cohesion
type: concept
tags: [fundamentals, design-principles, modularity]
status: complete
created: 2026-03-22
updated: 2026-03-22
---

# Coupling & Cohesion

Almost every architecture pattern — Hexagonal, Layered, Event-Driven — is trying to do the same two things: keep related code together, and keep unrelated code apart. Those two goals have names: cohesion and coupling.

---

## Cohesion

Cohesion is about what lives inside a module. High cohesion means everything in a class or module belongs there for the same reason. Low cohesion means one module is doing too many unrelated things.

```typescript
// Low cohesion — this class has no single reason to exist
class Utils {
  formatDate(d: Date): string { ... }
  sendEmail(to: string, body: string): void { ... }
  calculateTax(amount: number): number { ... }
}

// High cohesion — everything here is about pricing
class PricingCalculator {
  calculateTax(amount: number): number { ... }
  applyDiscount(amount: number, code: string): number { ... }
  formatPrice(amount: number, currency: string): string { ... }
}
```

The practical test: if you can't describe what a class does in one sentence, it probably does too much.

Low cohesion shows up as classes named `Manager`, `Helper`, `Utils`, or `Service` that have grown to cover whatever didn't fit elsewhere. They're hard to test because there's no clear boundary, and they keep changing because too many things depend on them.

---

## Coupling

Coupling is about what a module knows about other modules. Low coupling means a change to module A doesn't force changes to B, C, and D. High coupling means you can't touch anything without ripples.

There's a spectrum from worst to best:

| Type | What it means | Example |
|---|---|---|
| **Content** | Directly accessing another module's internals | Reading another class's private field |
| **Common** | Shared mutable global state | Global variables, singletons |
| **Control** | Passing a flag that controls behaviour | `process(order, isDryRun: true)` |
| **Stamp** | Passing a whole object when only part is needed | Passing `User` when you only need `userId` |
| **Data** | Passing only what's needed | `userId: string` |
| **Message** | Communicating through events | Publishing `OrderPlaced` |

Content and common coupling are bugs in disguise. Data and message coupling are what you're aiming for.

---

## Why they go together

You can't optimise one without thinking about the other. A class with high cohesion is easier to decouple because it has a clear, stable interface. A well-decoupled module is easier to keep cohesive because it isn't forced to know about everything around it.

Robert C. Martin put it simply:

> "Gather together things that change for the same reason. Separate things that change for different reasons."

That's both a definition of cohesion and a strategy for reducing coupling.

---

## What this looks like in practice

| Pattern | How it handles cohesion | How it handles coupling |
|---|---|---|
| Hexagonal Architecture | Domain logic grouped by use case | Ports decouple domain from infrastructure |
| CSR / Feature Slices | All feature code co-located | Slices stay independent by convention |
| Event-Driven Architecture | Producers own their events | Consumers decouple through the event bus |
| Layered Architecture | Each layer has one concern | Dependencies only flow in one direction |

---

## Smells worth knowing

**Low cohesion:**
- Class name contains "Utils", "Manager", "Helper", or "Misc"
- The class changes frequently for unrelated reasons
- You can't summarise what it does without a long list of "and also..."

**High coupling:**
- Changing one file requires editing several others
- Tests need extensive setup of things they're not testing
- A small bug fix ends up touching four different layers

---

## Further reading

- [Structured Design — Yourdon & Constantine (1979)](https://en.wikipedia.org/wiki/Structured_Design) — where the coupling taxonomy originally came from
- [Clean Architecture — Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) — applies these ideas at the architectural level
