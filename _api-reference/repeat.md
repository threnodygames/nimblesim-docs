---
title: repeat-times
layout: home
---

## `Repeat(int times)`

Repeats the last step a fixed number of times.

Example:

```csharp
  Nimble.Sim()
    .First(() => Debug.Log("Is there an echo in here?"))
    .Repeat(10)
```