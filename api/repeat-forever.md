## `RepeatForever()`

Repeats the last step indefinitely.

Examples:

```csharp
  Nimble.Sim()
    .First(PunchSelf())
    .Then(() => Debug.Log("Ow"))
    .RepeatForever()
```