---
title: sim builder
layout: home
---

NimbleSim uses the builder pattern & a fluent API to construct sequences that read like plain language. For example:

```csharp
Nimble.Sim()
  .First(MoveForward)
  .RepeatUntil(HitPellet)
  .Then(EatPellet)
  
```