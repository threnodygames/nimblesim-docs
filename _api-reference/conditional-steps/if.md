---
title: if
layout: home
parent: conditional-steps
---

## `If(...)`

---

Begins a conditional.

Supports:
  - lambda expression that returns a boolean
  - lambda expression with actor arg that returns a boolean
  - Nimble.Sim conditional action type
  - multiple lambda expressions that return a boolean
  - multiple lambda expressions with actor arg that return a boolean

Examples

```csharp
  Nimble.Sim()
    .If(() => health > 10)
      .Then(() => Debug.Log("1"))
      .Or(() => Debug.Log("2"))
    .Done();
```

```csharp
  Nimble.Sim()
    .If((GameObject actor) => actor.GetComponent<Health>().value > 10)
      .Then(() => Debug.Log("1"))
      .Or(() => Debug.Log("2"))
    .Done();
```

```csharp
  Nimble.Sim()
    .If(new CheckHealth())
      .Then(() => Debug.Log("1"))
      .Or(() => Debug.Log("2"))
    .Done();
```

```csharp
  Nimble.Sim()
    .If(() => health >= 10, () => health <= 20)
      .Then(() => Debug.Log("1"))
      .Or(() => Debug.Log("2"))
    .Done();
```

```csharp
  Nimble.Sim()
    .If((GameObject actor) => health >= 10, (GameObject actor) => health <= 20)
      .Then(() => Debug.Log("1"))
      .Or(() => Debug.Log("2"))
    .Done();
```

The result is a conditional branch builder supporting `.Then(...)` and then `.Or(...)`, both of which support the same arguments as `.Then(...)`.