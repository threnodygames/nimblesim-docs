## `Then(...)`

Adds a new step to the sequence. Effectively the same as `First(...)`.

Supports:

  - lambda expressions
  - lambda expressions with actor argument
  - NimbleSim action
  - NimbleSim sequence

Examples

```csharp
  Nimble.Sim()
    .First(() => Debug.Log("First"))
    .Then(() => Debug.Log("Then"))
```

```csharp
  Nimble.Sim()
    .First((GameObject actor) => Debug.Log(actor.tag))
    .Then((GameObject actor) => Debug.Log(actor.tag))
```

```csharp
   Nimble.Sim()
    .First(new ActionDebug())
    .Then(new ActionDebug())
```

```csharp
   Nimble.Sim()
    .First(
      Nimble.Sim().First(() => Debug.Log("Hello").Done())
    )
    .Then(
      Nimble.Sim().First(() => Debug.Log("Hello").Done())
    )
```