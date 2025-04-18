---
title: wait-until
layout: home
parent: circuit-breaker-steps
---

## `WaitUntil(...)`

---

Repeats a no-op action until a condition becomes true.

Accepts:
  - lambda expression that returns a boolean
  - lambda expression with actor arg that returns a boolean

Examples:

```csharp
  Nimble.Sim()
    .First(Sleep())
    .WaitUntil(() => energy >= 10)
    .Then(WakeUp())
    .Done();
```

```csharp
 Nimble.Sim()
    .First(Sleep())
    .WaitUntil((GameObject actor) => actor.GetComponent<Energy>().value >= 10)
    .Then(WakeUp())
    .Done();
```

Used for polling or gating further actions.