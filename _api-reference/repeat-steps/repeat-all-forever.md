---
title: repeat-all-forever
layout: home
parent: repeat-steps
---

## `RepeatAllForever()`

---

Repeats the entire sequence indefinitely.

Example

```csharp
  Nimble.Sim()
    .First(() => Debug.Log("Are we there yet"))
    .Then(() => Debug.Log("how long til we get there"))
    .Then(() => Debug.Log("is that it there?"))
    .Then(() => Debug.Log("are we there yet?"))
  .RepeatAllForever()
```