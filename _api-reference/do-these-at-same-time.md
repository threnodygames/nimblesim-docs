---
title: do-these-at-same-time
layout: home
---

## `DoTheseAtSameTime(...)`

Executes multiple actions or sequences simultaneously.

Supports
  - multiple NimbleSim.Action arguments
  - multiple NimbleSim.Sequence arguments
  - a combination of both

```csharp
  Nimble.Sim()
    .DoTheseAtSameTime(
      new Action.Debug("1"),
      new Action.Debug("2"),
      Nimble.Sim().First(() => Debug.Log("3")).Done()
    )
```

Useful for combining actions like moving and rotating at once.