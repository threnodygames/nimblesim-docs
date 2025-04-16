---
title: first
layout: home
---

## `First(...)`

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
```

```csharp
  Nimble.Sim()
    .First((GameObject actor) => Debug.Log(actor.tag))
```

```csharp
   Nimble.Sim()
    .First(new ActionDebug())
```

```csharp
   Nimble.Sim()
    .First(
      Nimble.Sim().First(() => Debug.Log("Hello").Done())
    )
```