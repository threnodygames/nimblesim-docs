---
title: while
layout: home
parent: parallel-steps
---

## `While()`

---

Begins a `While` block, allowing behavior to run while another sequence is active. Must be followed by `.Do(...)`.

Supports:

  - NimbleSim.Action
  - NimbleSim.Sequence

Examples:

```csharp
  Nimble.Sim()
    .While(new ActionRunAway())
      .Do(() => Debug.Log("AAAAAAAAA"))
    .Done();
```

```csharp
  Nimble.Sim()
    .While(Nimble.Sim().First(new ActionRunAway()).Done())
      .Do(() => Debug.Log("AAAAAAAAA"))
    .Done();
```

`Do()` supports the same arguments as `.Then()`