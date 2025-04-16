## `WaitUntil(...)`

Repeats a no-op action until a condition becomes true.

Accepts:
  - lambda expression that returns a boolean
  - lambda expression with actor arg that returns a boolean

Examples:

```csharp
  Nimble.Sim()
    .First(Sleep())
    .WaitUntil(() => energy >= 10)
    .Then(WakeUp())
```

```csharp
 Nimble.Sim()
    .First(Sleep())
    .WaitUntil((GameObject actor) => actor.GetComponent<Energy>().value >= 10)
    .Then(WakeUp())
```

Used for polling or gating further actions.