---
title: repeat-for-seconds
layout: home
---

## `RepeatForSeconds(float seconds)`

Repeats the last step for a fixed duration in seconds.

Example:

```csharp
  Nimble.Sim()
    .First(() => Debug.Log("Waaaa"))
    .RepeatForSeconds(20391248)
```