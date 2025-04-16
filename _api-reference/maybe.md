## `Maybe(...)`

Adds a step that runs with a given chance (default=0.5).

Supports:
  - lambda expressions
  - lambda expressions with actor argument
  - NimbleSim action
  - NimbleSim sequence

Examples

```csharp
  Nimble.Sim()
    .Maybe(() => Debug.Log("First"))

  Nimble.Sim()
    .Maybe(() => Debug.Log("First"), 0.3)
```

```csharp
  Nimble.Sim()
    .Maybe((GameObject actor) => Debug.Log(actor.tag))

  Nimble.Sim()
    .Maybe((GameObject actor) => Debug.Log(actor.tag), 0.3)
```

```csharp
   Nimble.Sim()
    .Maybe(new ActionDebug())

   Nimble.Sim()
    .Maybe(new ActionDebug(), 0.3)
```

```csharp
   Nimble.Sim()
    .Maybe(Nimble.Sim().First(() => Debug.Log("1")).Done())

    Nimble.Sim()
    .Maybe(Nimble.Sim().First(() => Debug.Log("1")).Done(), 0.3)
```