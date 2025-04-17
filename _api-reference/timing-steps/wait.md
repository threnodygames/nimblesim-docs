---
title: wait
layout: home
parent: timing-steps
---

## `Wait(float seconds)`

Pauses the sequence for a number of seconds.

Example:

```csharp
  Nimble.Sim()
    .First(() => Debug.Log("Start"))
    .Wait(3)
    .Then(() => Debug.Log("Three seconds later..."))
  .Done();
```