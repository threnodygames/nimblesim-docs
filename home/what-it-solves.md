---
title: why nimblesim?
layout: home
nav_order: 0
parent: home
---

## 🎯 Why NimbleSim?

Ever find yourself writing code like this?

```csharp
// Bee foraging behaviour
 if (IsNighttime())
    {
      if (IsAtHive())
      {
        Sleep();
      }
      else
      {
        ReturnToHive();
      }

      return;
    }

    if (AlreadyHasNectar())
    {
      if (IsAtHive())
      {
        StoreNectar(flower)
      }
      else
      {
        FlyToHive();
      }

      return;
    }

    if (!IsNearFlower())
    {
      flower = FlyToNextFlower();
      return;
    }

    if (flower.IsNotOccupied() && flower.HasNectar())
    {
      flower = FlyToNextFlower();
      return;
    }

    Harvest(flower);
```

You try to make it readable by writing meaningful method names, implementing early returns so that you avoid deep if / else nesting and unnecessary checks.

But this is 35 lines of code, not counting the whitespace. And despite your best efforts, it still has some overhead for understanding, it probably takes some work to add additional behaviours, and extracting parts of this behaviour to share with other actors either involves external methods that will need to be created and attached to this object, or some copy / pasting.

*This is the same code in NimbleSim.*

```csharp
 Sequence tryHarvestFlower = Nimble.Sim()
    .If(flower.IsNotOccupied, flower.HasNectar)
        .Then(_ => Harvest(flower))
        .Or(Action.Nothing())
    .Done();

Sequence forage = Nimble.Sim()
    .If(IsNearFlower)
        .Then(tryHarvestFlower)
        .Or(_ => flower = FlyToNextFlower())
    .Done();

beeBehaviour = Nimble.Sim()
    .If(AlreadyHasNectar)
        .Then(ReturnToHive)
        .Or(forage)
    .RepeatUntil(IsNightTime)
    .Then(ReturnToHive)
    .Then(Sleep)
.Done();
```

All achieved in only 18 lines of code that read just like a story.

To summarise, NimbleSim is:

- 🔍 **Readable** – behaviour scripts read like little stories.
- 🧱 **Composable** – simple atomic actions can be glued together to create layered, reactive behaviours.
- 🧠 **Declarative** – you say *what* the behaviour should do, not *how* to do it.
- 🌀 **Interruptible & Reactive** – behaviours can respond to changes in the world mid-flow.
- 🧰 **Plain C#** – no custom editor or node graph required.