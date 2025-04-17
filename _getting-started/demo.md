---
title: demo
layout: home
nav_order: 1
---

# Demo Project

## Loading the project

Go to `NimbleSim/Demo/Scenes` and load the `NimbleSimMovementDemo` scene.

## Summary

The demo simulates the following behaviour:

- Agent (capsule) randomly selects one of the red cubes to run to
- When it reaches the cube, it has a random chance to fall over and take some damage
- If it falls over, it lays down for 1 second, takes some damage and then gets back up
- Whole sequence repeats until the agent's health reaches 0

In parallel, another sequence is being run on the camera which makes it follow the agent.

## Explanation

### Sequences

NimbleSim uses sequences & actions to orchestrate behaviours. For example, we can create a sequence to simulate an actor falling down and taking damage:

```csharp
  // NimbleMovement.cs : line 33
  Sequence FalloverAndTakeDamage()
  {
      void FallOver(GameObject actor)
      {
          actor.transform.rotation = Quaternion.Euler(Vector3.right * 90);
      }

      return Nimble.Sim()
          .First(FallOver)
          .Wait(1)
          .Then(() => health -= 1)
      .Done();
  }
```

Nimble aims to read as close to plain-english as possible, so (hopefully) it's clear that this sequence is saying:

- Fall down
- Wait for 1 seconds
- Then take damage

Similarly, we can simulate randomly choosing an object to run to:

```csharp
// NimbleMovement.cs : line 48
 Sequence RunToRandomObject()
{
    return Nimble.Sim()
        .RandomlyDoOneOf(
            moveTo(objectOne),
            moveTo(objectTwo),
            moveTo(objectThree),
            moveTo(objectFour)
        )
    .Done();
}
```

Sequences are also composable, meaning we can take the previous sequences and compose them into a larger sequence. Then we can add guards, such as `RepeatPreviousUntil` to control execution.

```csharp
// NimbleMovement.cs : line 63
all = Nimble.Sim()
    .First(RunToRandomObject())
    .Maybe(FalloverAndTakeDamage(), 0.4f)
    .RepeatPreviousUntil(() => health <= 0)
    .Then(actor => Destroy(actor))
.Done();
```

Now, the actor will run around and *maybe* fall over repeatedly. When it's health hits zero, the circuit breaker will kick in, and progress to the `Then` step, destroying the actor.

Sequences can run on any actor, regardless of which class they were created in. For example, we can create a sequence to have the camera follow our capsule:

```csharp
// NimbleMovement.cs : line 70
cameraFollow = Nimble.Sim()
    // CameraFollow is an action. More on these in a second.
    .First(new CameraFollow(this.gameObject))
.RepeatForever().Done();
```

Now, we can run both the movement sequence and the camera sequence, in parallel, on our capsule:

```csharp
// NimbleMovement.cs : line 75
void Update()
{
    all.Update(gameObject);

    cameraFollow.Update(cam);
}
```

By passing in a game object to update, you control which actor the sequence will operate on.

### Actions

Actions are units of behaviour. While you can absolutely build sequences out of lambdas alone, actions give you more fine-grained control over execution. The movement of the agent is orchestrated by the following action:

```csharp
  // MoveTo.cs
  public class MoveTo : Action
  {
    Transform destination;
    int speed = 10;

    public MoveTo(Transform destination)
    {
      this.destination = destination;
    }

    public override void Update(GameObject actor)
    {
      actor.transform.position = Vector3.MoveTowards(
        actor.transform.position,
        destination.position,
        speed * Time.deltaTime
      );
    }

    protected override bool IsComplete(GameObject actor)
      => actor.transform.position == destination.position;

    public override Action Get() => new MoveTo(destination);
}
```

By encapsulating this behaviour in an action, you can reuse it throughout your sequences. It also means we can define a clear termination condition (the `IsComplete` method) which tells the sequence when to proceed.

If we tried to do this with lambdas, we would need to add an additional circuit breaker to check if the actor has reached the destination. We would also have to extract this logic to some other place anyway if we wanted to reuse it for other actors, and manually invoke the update and IsComplete methods.

Wrapping this behaviour in an action and passing it into a sequence means NimbleSim will handle all of this.

## Summary

With only these 4 sequences, 2 actions & ~140 lines (including whitespace, using statements and class setup), it was possible to simulate:

- Random choice
- Circuit breakers
- Timing (wait behaviour)
- Camera follow
- Movement
- Taking damage
- Death

All described in language that reads like a story.

To dive deeper into NimbleSim, consider checking out:

- [the beehive simulation example]({{ site.baseurl }}/examples/beehive-sim)
- [the tutorial]() (coming soon)
- [the api reference]({{ site.baseurl }}/api-reference/api)


