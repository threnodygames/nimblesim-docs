---
title: wait
layout: home
---

## `Wait(float seconds)`

Pauses the sequence for a number of seconds.

Example:

```csharp
  Nimble.Sim()
    .First(() => Debug.Log("Start"))
    .Wait(3)
    .Then(() => Debug.Log("Three seconds later..."))
```