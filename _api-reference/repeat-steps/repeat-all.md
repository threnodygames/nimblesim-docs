---
title: repeat-all
layout: home
parent: repeat-steps
---

## `Sequence RepeatAll(int times)`

---

Repeats the entire sequence a fixed number of times.

Example:

```csharp
  Nimble.Sim()
    .First(StandAtMirror())
    .Then(() => Debug.Log("Bloody Mary"))
    .RepeatAll(3);
```