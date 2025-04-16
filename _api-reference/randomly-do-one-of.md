---
title: randomly-do-one-of
layout: home
---

## `RandomlyDoOneOf(...)`

Executes one random action or sequence.

Supports

- array of lambda expressions
- array of lambda expressions with actor argument
- array of Nimble.Action type
- array of Nimble.Sequence type
- array of (Nimble.Action, float weight)
- array of (Nimble.Sequence, float weight)

Examples:

```csharp
  Nimble.Sim()
    .RandomlyDoOneOf(
      () => Debug.Log("1"),
      () => Debug.Log("2")
    )
```

```csharp
  Nimble.Sim()
    .RandomlyDoOneOf(
      (GameObject actor) => Debug.Log($"1: {actor.tag}"),
      (GameObject actor) => Debug.Log($"2: {actor.tag}")
    )
```

```csharp
  Nimble.Sim()
    .RandomlyDoOneOf(
      new ActionDebug("1"),
      new ActionDebug("2")
    )
```

Use weighted form for probabilistic behavior:

```csharp
  Nimble.Sim()
    .RandomlyDoOneOf(
      (new ActionDebug("1"), 0.3),
      (new ActionDebug("2"), 0.7)
    )
```