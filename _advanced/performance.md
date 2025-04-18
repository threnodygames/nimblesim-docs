# Performance

`Action.Get()` is provided primarily to avoid state leakage, but it does potentially create a performance bottleneck. This is because each iteration of a sequence will call the method & create a new allocation.

If this is affecting your game, you can opt to return `this` from `Get()` and then manually manage state inside the `OnStart()` hook.