---
title: villager-metabolism
parent: village-sim
nav_order: 1
layout: home
---

# Villager Metabolism

## Prefab and Script Setup

1. First, create a new capsule object in your scene and name it 'Villager'
2. Create a script in `World/Villager` called `Villager.cs` and drag it onto your capsule

## Villager Metabolism

Villagers will experience hunger and thirst, and if either of these are neglected for too long, they will die.

Here's a video of what you'll have by the end of this step:

<iframe width="800" height="450"
  src="https://www.youtube.com/embed/faf943fZFMM"
  title="NimbleSim Bee Demo"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
  allowfullscreen
  style="border-radius: 12px; margin-top: 1rem;">
</iframe>


### Story

Because NimbleSim lets us script behaviours in plain-language, we can write a "story" for our villager's metabolism and then copy it over. Here's what that looks like:

```csharp
Wait(3)
Then(IncreaseHunger)
Then(IncreaseThirst)
RepeatPreviousUntil(Dying)
Then(Die)
```

IncreaseHunger, IncreaseThirst, Dying and Die are all simple, 1 line methods:

```csharp
  void IncreaseHunger() => hunger += 1;
  void IncreaseThirst() => thirst += 1;
  bool Dying() => thirst > 10 || hunger > 20;
  void Die(GameObject actor) => Object.Destroy(actor);
```

This is all we need for now. 

## Implementation

Sequences in NimbleSim are completely independent of Unity objects — they’re just behavior blueprints. That means we can write a Metabolism class that builds a sequence and hand it off to any villager when they need it.

This approach also opens up the possibility of adding other creatures with metabolisms later, like animals.

For now though, we'll only be supporting villagers, so let's create a `Metabolism.cs` class in the `Villager/` folder and add our hunger and thirst properties.

{: .note}
This class doesn't need to inherit from MonoBehaviour, since it doesn’t need to live in the scene or hook into Unity’s lifecycle.

```csharp
public class Metabolism
{
  int hunger = 0;
  int thirst = 0;
}
```

Also copy over the methods we'll use in our sequence:

```csharp
  void IncreaseHunger() => hunger += 1;
  void IncreaseThirst() => thirst += 1;
  bool Dying() => thirst > 10 || hunger > 20;
  void Die(GameObject actor) => Object.Destroy(actor);
```

Now add a method which returns the sequence:

```csharp
   public Sequence Get()
  {
    return Nimble.Sim()
      .Wait(3)
      .Then(IncreaseHunger)
      .Then(IncreaseThirst)
      .RepeatPreviousUntil(Dying)
      .Then(Die)
    .Done();
  }
```

You can optionally add Debug statements if you want to track what's happening:

```csharp
return Nimble.Sim()
  .Wait(3)
  .Debug("Increasing...")
  .Then(IncreaseHunger)
  .Then(IncreaseThirst)
  .Then(() => Debug.Log($"New hunger: {hunger}"))
  .Then(() => Debug.Log($"New hunger: {thirst}"))
  .RepeatPreviousUntil(Dying)
  .Then(Die)
  .Done();
```

{: .note }
We use `Then` with a lambda expression for the hunger and thirst logs because plain strings are evaluated immediately when the sequence is built. Using a `Func<string> (like () => $"Hunger: {hunger}")` ensures the log message is re-evaluated each time the step runs, reflecting the updated values.


Now, go back to your villager class and add the following in the class body:

```csharp
Sequence metabolism;

void Start()
{
    metabolism = new Metabolism().Get();
}

void Update()
{
    metabolism.Update(gameObject);
}
```

This is all you need to activate the metabolism. If we later wanted to add other creatures with metabolisms, we would only need to add those three lines. Arguments could also be added to the `Metabolism` class to customise starting values & threshold values, e.g.:

```csharp
public Metabolism(int startingHunger, int startingThirst, int maxThreshold)
```

Now, if you switch back to your editor, make sure the `Villager` script is attached to the capsule & hit play, you should see the hunger and thirst ticking up.

Head here to see the [full code](https://github.com/threnodygames/nimblesim-village/tree/main/villager-setup) for these two files.
