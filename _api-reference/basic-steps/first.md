---
title: first
layout: home
nav_order: 0
parent: basic-steps
---

## `First(...)`

---

Initializes the sequence with a first step.

Supports:

  - lambda expressions
  - lambda expressions with actor argument
  - NimbleSim action
  - NimbleSim sequence

Examples

```csharp
  Nimble.Sim()
    .First(() => Debug.Log("First"))
  .Done();
```

```csharp
  Nimble.Sim()
    .First((GameObject actor) => Debug.Log(actor.tag))
  .Done();
```

```csharp
   Nimble.Sim()
    .First(new ActionDebug())
  .Done();
```

```csharp
   Nimble.Sim()
    .First(
      Nimble.Sim().First(() => Debug.Log("Hello").Done())
    )
  .Done();
```