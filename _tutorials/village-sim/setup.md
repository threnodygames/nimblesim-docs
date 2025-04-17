---
title: setup
parent: village-sim
nav_order: 0
layout: home
---

# Setup

## Installation and folder structure

1. Create a new Unity Project and import `NimbleSim`
2. Create a folder in `Assets/` to store your game assets (mine is `World/`)
3. Create an empty game object and call it `Village`
4. Create a folder in `World/` called `Village/` and add a `Scripts` folder to `Village/`

## Verify NimbleSim is installed and working

1. Create one final folder in `Village/` called `Scripts/` and add a new MonoBehaviour script called `Village`

![]({{ site.baseurl }}/assets/tutorials/village-sim/setup/add-village-script.png)

2. Now we can check if NimbleSim is setup and working correctly.

NimbleSim uses sequences of actions or lambda expressions to script behaviour in plain-language. We'll create a sequence now to check it's all working correctly.

3. Create a new Sequence object at the top of your `Village.cs` script. We can just call it test:

```csharp
  Sequence test;
```

Then, in the start method, we can build the sequence:

```csharp
 void Start()
    {
        test = Nimble.Sim()
            .Debug("Working")
            .Wait(2)
        .RepeatAll(3);
    }
```

This sequence should print "Working" to the console, wait 2 seconds and then repeat 3 times (for a total of 4).

Run the sequence by adding this line to your `Village.cs` `Update()` method:

```csharp
   void Update()
    {
        test.Update(gameObject);
    }
```

`Village.cs` should now look like this:

```csharp
using UnityEngine;
using NimbleSim;

public class Village : MonoBehaviour
{
    Sequence test;

    void Start()
    {
        test = Nimble.Sim()
            .Debug("Working")
            .Wait(2)
        .RepeatAll(3);
    }

    void Update()
    {
        test.Update(gameObject);
    }
}
```

To test this, go back to the editor and drag the `Village.cs` script onto the `Village` game object, then hit play.

If you see `Working` printed 4 times, with a 2 second delay in between, then NimbleSim is all good to go and we can get started.

Next: [Villager metabolism]({{ site.baseurl }}/tutorials/village-sim/villager-metabolism)