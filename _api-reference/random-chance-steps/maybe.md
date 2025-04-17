---
title: maybe
layout: home
parent: random-chance-steps
---

## `Maybe(...)`

Adds a step that runs with a given chance (default=0.5).

Supports:
  - lambda expressions
  - lambda expressions with actor argument
  - NimbleSim action
  - NimbleSim sequence

Examples

```csharp
  Nimble.Sim()
    .Maybe(() => Debug.Log("First"))
  .Done();

  Nimble.Sim()
    .Maybe(() => Debug.Log("First"), 0.3)
  .Done();
```

```csharp
  Nimble.Sim()
    .Maybe((GameObject actor) => Debug.Log(actor.tag))
  .Done();

  Nimble.Sim()
    .Maybe((GameObject actor) => Debug.Log(actor.tag), 0.3)
  .Done();
```

```csharp
   Nimble.Sim()
    .Maybe(new ActionDebug())
  .Done();

   Nimble.Sim()
    .Maybe(new ActionDebug(), 0.3)
  .Done();
```

```csharp
   Nimble.Sim()
    .Maybe(Nimble.Sim().First(() => Debug.Log("1")).Done())
  .Done();

  Nimble.Sim()
    .Maybe(Nimble.Sim().First(() => Debug.Log("1")).Done(), 0.3)
  .Done();
```