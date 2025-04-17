---
  title: eating-and-drinking
  nav_order: 2
  parent: village-sim
  layout: home
---

# Eating and Drinking

Currently our villagers have no way to address their hunger or thirst. We need to develop a system whereby they can react to these stats getting too high, go to the food hall, get what they need, and then update their metabolism. Just to make it harder, let's also say they have to wait at the food hall for 2 seconds before updating anything to simulate a meal. On top of this, we need to support the scenario where the food hall has no stock. For now, if this happens, we'll just make the villager wait until there is stock, BUT they shouldn't ask again until that point.

## What we could do

In order to achieve the behaviour described above, we could:

- Make hunger and thirst public or expose getters like GetHunger() and GetThirst()
- Add SatisfyHunger() / SatisfyThirst() methods to the metabolism so the villager can update these after eating
- Build a state machine or conditional logic that:
  - checks hunger/thirst levels
  - chooses where to go
  - checks if food hall has stock
  - eats if food hall has stock
  - waits for 5 seconds and then asks again if it doesn't
- Simulate eating with coroutines or timers
- Add a methods on the food hall like Take(), HasStock()

This approach does work, but:
  - it's complex, lengthy and brittle
  - it involves writing all this behaviour onto `Villager.cs` which make sharing all or some of the behaviour with other actors difficult
  - it separates the purpose of the food hall from the food hall. this is fine with one entity type, but what if we later wanted to add traders? invaders? aliens? each of these could have a metabolism and each could want to use the food hall, but that interaction lives on the villager, so now they have to reproduce it. You could create a base class to share that behaviour, but then what if the invader shouldn't wait around when the food hall is empty and should instead burn it down? And all of these questions are about just the food hall, what about the other structures we're going to create?

Instead of all that, we're going to use NimbleSim to create this behaviour in 10 minutes. No state machines, no ifs, no coroutines. In fact, we will only need to add about 2 lines to the villager itself.

But before all that, let's give the villagers somewhere to eat.

## Food Hall

Create a food hall and place it somewhere in the scene. I'm going to use a simple cube with a brownish material.

Add a cube for the ground and stretch it, then place another cube on top called 'Food Hall'.

![]({{ site.baseurl }}assets/tutorials/village-sim/food-and-drink/food-hall-and-ground.png)

## FoodHall.cs

### Story

The story of the food hall will live on the food hall itself. Actors will "ask" the food hall how it should be used.

We'll use just two short sequences for this:

```
First(ReduceFood)
Then(ReduceDrink)
Then(GiveFood)
Done()

First(ComeHere)
If(HaveStock)
  Then(Take)
  Or(Refuse)
.Done()
```

### Implementation

Create the FoodHall.cs MonoBehaviour script.

```
Assets -> World -> Village -> Structures -> FoodHall -> FoodHall.cs
```

I might overuse folders slightly so just put it wherever makes sense to you.

Now, add the food and drink properties:

```csharp
    int food = 1;
    int drink = 1;
```

And a method called `Use`:

```csharp
public Sequence Use(Sequence OnFoodRefused, Sequence OnFoodGiven)
{
    var GiveFood = Nimble.Sim()
      .First(() => food -= 1)
      .Then(() => drink -= 1)
      .Then(OnFoodGiven)
    .Done();

    return Nimble.Sim()
      .First(() => Debug.Log("Come here: stubbed out for now."))
      .If(() => food > 0 && drink > 0)
          .Then(GiveFood)
          .Or(OnFoodRefused)
      .Done();
}
```

Minus the `ComeHere` step (which we'll do after the other stuff), this is all we need.

Whenever our villager (or any actor) needs to use the food hall, they just ask it and receive instructions. And changing this behaviour later is easy because it's only defined in once place.

We also inject two sequences, `OnFoodGiven` and `OnFoodRefused`. Providing these means:

- the food hall only cares about it's own behaviour
- any actor who uses the food hall can receive it's instructions but still decide how it will react in each circumstance

## Metabolism.cs

Now that we have the food hall setup, we need a way to tell the actor that they should go there when it's necessary. 

Like the food hall, the metabolism shouldn't care about which actor it's attached to or what they do when they're hungry / thirsty.

***The actor also shouldn't have to ask 'am i hungry?', it should just feel the hunger and react.***

At the same time, though, the actor needs to be able to suppress their metabolism, for example in the case where there is no food available.

Put together, this is a complex chain of behaviour, especially when each class is guarding its own internals, but we can achieve it with Sequences and composition.

First, add a new sequence inside Get, just above the return:

```csharp
Sequence CheckNeeds = Nimble.Sim()
  .If(() => hunger >= 7 || thirst >= 3)
    .Then(() => Debug.Log("Help"))
    .Or(Action.Nothing())
  .RepeatAllForever();
```

And extract the first 3 steps of the original sequence:

```csharp
.Wait(10f)
.Then(IncreaseHunger)
.Then(IncreaseThirst)
```

Out into a new sequence:

```csharp
Sequence BodyClock = Nimble.Sim()
  .Wait(10f)
  .Then(IncreaseHunger)
  .Then(IncreaseThirst)
.RepeatAllForever();
```

Finally, add a `DoTheseAtSameTime` step to the returned sequence:

```csharp
return Nimble.Sim()
  .DoTheseAtSameTime(BodyClock, CheckNeeds)
  .RepeatPreviousUntil(Dying)
  .Then(Die)
  .Done();
```

This is what your Get() method should look like now:

```csharp
  public Sequence Get()
  {
    Sequence CheckNeeds = Nimble.Sim()
      .If(() => hunger >= 7 || thirst >= 3)
        .Then(() => Debug.Log("Help"))
        .Or(Action.Nothing())
      .RepeatAllForever();

    Sequence BodyClock = Nimble.Sim()
        .Wait(10f)
        .Then(IncreaseHunger)
        .Then(IncreaseThirst)
      .RepeatAllForever();

    return Nimble.Sim()
      .DoTheseAtSameTime(BodyClock, CheckNeeds)
      .RepeatPreviousUntil(Dying)
      .Then(Die)
      .Done();
  }
```

When the villager executes this sequence, it will now check the hunger and thirst levels of its metabolism and log out "Help" when it's too low, *unless* the villager is suppressing their needs.

This means that without even opening the Villager class, we've been able to give it new behaviour.

Now we need to make the villager use the Food Hall when hungry or thirsty.

```csharp
  .If(() => hunger >= 7 || thirst >= 3)
    // This should trigger a food hall visit
    .Then(() => Debug.Log("Help"))
    .Or(Action.Nothing())
  .RepeatAllForever();
```

But we want to do that without creating a coupling between `FoodHall.cs` and `Metabolism.cs` because we might want to give other actors, like animals, a metabolism too.

So similar to the food hall, let's just inject the sequence that should happen when hungry:

```csharp
  public Sequence Get(Sequence HandleMetabolicNeeds)
```

and pass it to the `Then()` arm of the `If()`.

Also add another sequence to handle resetting both hunger and thirst after the needs have been handled. 

Now, the `Get()` method should look like this:

```csharp
  public Sequence Get(Sequence HandleMetabolicNeeds)
  {
    Sequence CheckNeeds = Nimble.Sim()
      .If(() => hunger >= 7 || thirst >= 3)
        .Then(HandleMetabolicNeeds)
        .Or(Action.Nothing())
      .RepeatAllForever();

    Sequence BodyClock = Nimble.Sim()
        .Wait(10f)
        .Then(IncreaseHunger)
        .Then(IncreaseThirst)
      .RepeatAllForever();

    return Nimble.Sim()
      .DoTheseAtSameTime(BodyClock, CheckNeeds)
      .RepeatPreviousUntil(Dying)
      .Then(Die)
      .Done();
  }
```

Now, let's add a `Suppress()` method which allows the actor to ignore their metabolism for a short time. 

```csharp
bool suppressed = false;

public Sequence Suppress(float time)
{
  return Nimble.Sim()
      .First(() => suppressed = true)
      .Wait(time)
      .Then(() => suppressed = false)
    .Done();
}
```

Next, update `CheckNeeds` with the new condition:

```csharp
Sequence CheckNeeds = Nimble.Sim()
  .If((() => hunger >= 7 || thirst >= 3) && !suppressed)
    .Then(HandleMetabolicNeeds)
    .Or(Action.Nothing())
  . RepeatAllForever();
```

We can also extract out the first condition to make this more readable.

```csharp
bool ShouldHandleMetabolicNeeds() => hunger >= 7 || thirst >= 3;
```

```csharp
Sequence CheckNeeds = Nimble.Sim()
  // If can take in multiple lambda expressions.
  .If(ShouldHandleMetabolicNeeds(), () => !suppressed)
    .Then(HandleMetabolicNeeds)
    .Or(Action.Nothing())
  .RepeatAllForever();
```

And finally, add a `Replenish()` method which resets hunger and thirst:

```csharp
  public Sequence Replenish()
  {
    return Nimble.Sim()
        .First(() => hunger = 0)
        .Then(() => thirst = 0)
      .Done();
  }

```

## Villager.cs

With both the FoodHall and the Metabolism set up, all the villager has to do is connect them. 

### Implementation

Add a field for FoodHall:

```csharp
[SerializeField]
FoodHall foodHall;
```

Now, create Sequence-returning Method on the villager called `OnFoodRefused`:

```csharp
Sequence OnFoodRefused()
  {
     return metabolism.Suppress(5);
  }
```

Now, create Sequence-returning Method on the villager called `OnFoodGiven`:

```csharp
Sequence OnFoodGiven()
  {
      return metabolism.Replenish();
  }
```

Next, we can call `foodHall.Use()` and pass in these two methods. 

```csharp
Sequence HandleMetabolicNeeds = foodHall.Use(OnFoodRefused(), OnFoodGiven());
```

And then pass that into their metabolism.Get() method:

```csharp
metabolism = new Metabolism().Get(HandleMetabolicNeeds);
```

Minus the code that makes the actor walk to the food hall, that's it. 

If you want to see this working right now, you should:

- make sure the FoodHall script is attached to the food hall
- make sure to drag the FoodHall game object onto the capsule
- make the `FoodHall` food and drink fields serializable so you can watch them
- reduce the hunger or thirst threshold in `Metabolism.cs`, line 17 (`.If(() => hunger >= 7 || thirst >= 1)`) so that it doesn't take ages to trigger
- maybe add a few Debug statements to the sequences, e.g.:

```csharp
    // FoodHall.cs
    Sequence Take()
    {
        return Nimble.Sim()
            .First(() => food -= 1)
            .Then(() => drink -= 1)
            .Then(actor => Debug.Log($"{actor.tag}: Eating"))
            .Wait(3)
        .Done();
    }
```

You should be able to see that everything is working as expected. 

## Movement logic

The last step of this section involves making the actor move. To keep it simple (for now), we'll just use some simple movement.

`FoodHall.cs` owns the logic that describes how to get to it, but that class doesn't have any knowledge of the actor who should move towards it. NimbleSim provides a handle to the actor running the sequence via lambda expressions, so you could do something like this:

```csharp
 return Nimble.Sim()
  .First(actor =>
  {
      actor.transform.position = Vector3.MoveTowards(
          actor.transform.position,
          transform.position,
          3.0f * Time.deltaTime
      );
  })
  .RepeatUntil(actor => Vector3.Distance(actor.transform.position, transform.position) < 4.0f)
  .If(() => food > 0 && drink > 0)
      .Then(Take())
      .Or(Refuse)
  .Done();
```

And this does work, but it's messy and brittle & would need to be duplicated for other structures like `FoodHall` which want to call actors.

Also we needed to add the `RepeatUntil` step because otherwise the sequence would move the actor for 1 frame and then progress.

So instead of all that, we can create a NimbleSim `Action`.

## Goto.cs

NimbleSim actions are self-contained units of behaviour that can be composed into sequences & give you precise control over how they're executed.

{: .note }
You can learn more about Actions [here]({{ site.baseurl }}/concepts/actions).

### Implementation

Create a new folder under `Villager/` called `Actions/` and then add a `Goto.cs` file.

Actions need to inherit from the `Action` type provided by NimbleSim, so your class declaration should look like this:

```csharp
using NimbleSim;

public class Goto : Action {
  
}
```

All actions need to implement a `Get()` method that returns a new instance of itself, so add this:

```csharp
  // Read more about this in the action docs
  public override Action Get()
  {
    return new Goto();
  }
```

We'll need some way of storing the destination we want the actor to go to, so we can add a property for that and a constructor parameter. We'll also need to update the `Get()` method to pass the property into the new instance:

```csharp
public class Goto : Action
{
  Vector3 destination;

  public Goto(Vector3 destination)
  {
    this.destination = destination;
  }

  public override Action Get()
  {
    return new Goto(destination);
  }
}
```

This is all the boilerplate we need. For the actual behaviour, we'll need to add two overrides:

```csharp
  public override void Update(GameObject actor)
  {

  }

  protected override bool IsComplete(GameObject actor)
  {

  }
```

`Update` is run every frame, similar to a regular `MonoBehaviour`.
`IsComplete` is how the Sequence knows the action is done & it can move to the next step.

The last thing we need to do is add the movement code & the complete condition:

```csharp
  public override void Update(GameObject actor)
  {
      actor.transform.position = Vector3.MoveTowards(
          actor.transform.position,
          transform.position,
          3.0f * Time.deltaTime
      );
  }

  protected override bool IsComplete(GameObject actor)
  {
    return Vector3.Distance(actor.transform.position, destination) < 4.0f;
  }
```

## FoodHall.cs

Now `Use()` in `FoodHall.cs` becomes this:

```csharp
  public Sequence Use(Sequence Refuse)
    {
      return Nimble.Sim()
        .First(new Goto(transform.position))
        .If(() => food > 0 && drink > 0)
            .Then(Take())
            .Or(Refuse)
        .Done();
    }
```

NimbleSim handles executing and terminating the action, so this is all you have to do.

## Testing

Jump back to the editor and hit play. Your villager should now move to the food hall when hungry and try to eat or drink. If no food / water is available, your villager should suppress its metabolism.

To make testing easier, you can:

Add a log to the `CheckNeeds` sequence inside of `Metabolism.cs`:

```csharp
   Sequence CheckNeeds = Nimble.Sim()
      .First(() => Debug.Log("Suppressed: " + suppressed))
```

Reduce the `thirst` threshold to one in the same class:

```csharp
  bool ShouldHandleMetabolicNeeds() => hunger >= 7 || thirst >= 1;
```

Also add some logs to the `Suppress` and `Replenish` methods:

```csharp
  public Sequence Suppress(float time)
  {
    return Nimble.Sim()
        .First(() => suppressed = true)
        .Debug("Metabolism suppressed")
        .Wait(time)
        .Debug("Metabolism active")
        .Then(() => suppressed = false)
      .Done();
  }

  public Sequence Replenish()
  {
    return Nimble.Sim()
        .First(() => hunger = 0)
        .Then(() => thirst = 0)
        .Then(() => Debug.Log($"Replenished: Hunger: {hunger} | Thirst: {thirst}"))
      .Done();
  }
```

Add a log to the `GiveFood` sequence of `FoodHall`:

```csharp
  var GiveFood = Nimble.Sim()
      .First(() => food -= 1)
      .Then(() => drink -= 1)
      .Then(OnFoodGiven)
      .Then(() => Debug.Log($"Food available: {food}, Drink available: {drink}"))
  .Done();
```