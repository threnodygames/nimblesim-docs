---
title: repeat-until
layout: home
parent: circuit-breakers
---

## `RepeatUntil(...)`

Repeats the last step until a condition becomes true.

Supports:

- lambda function returning boolean
- lambda function with actor arg returning boolean

Examples:

```csharp
  Nimble.Sim()
    .First(() => Debug.Log("Write docs"))
    .RepeatUntil(Dead())
```

```csharp
  Nimble.Sim()
    .First((GameObject actor) => actor.GetComponent<Fingers>().Write())
    .RepeatUntil(Crying())
```