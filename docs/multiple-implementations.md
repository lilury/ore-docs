---
sidebar_position: 3
---

# Multiple Implementations

In Ore, you could also register multiple implementations of the same service (ie. interface)

```go
//simpleCounter
ore.RegisterFunc[Counter](ore.Scoped, func(ctx context.Context) (Counter, context.Context) {
    return &simpleCounter{}, ctx
})

//atomicCounter
ore.RegisterFunc[Counter](ore.Scoped, func(ctx context.Context) (Counter, context.Context) {
    return &atomicCounter{}, ctx
})

//quantumCounter
ore.RegisterFunc[Counter](ore.Transient, func(ctx context.Context) (Counter, context.Context) {
    return &quantumCounter{}, ctx
})

//rgbCounter
ore.RegisterCreator[Counter](ore.Singleton, &rgbCounter{})
```

here we've registered 4 different implementations for `Counter` service in all ways and lifetimes possible.

now let's resolve `Service` using `ore.GetList`:

```go
allCounters, ctx := ore.GetList[Counter](context.Background())

fmt.Println(len(allCounters)) // outputs 4

for _, counter := range allCounters {
    fmt.Println(counter.AddOne())
}
```

resolving only one implementation:

```go
counter, ctx := ore.Get[Counter](context.Background())
```

you'll notice that `counter` is of type `rgbCounter{}`, because by default, Ore will resolve only the last registered
implementation for `Service` when `ore.Get` is called.


## Multiple Keyed Implementations

You could also register multiple keyed implementations of the same service :

```go
//simpleCounter
ore.RegisterKeyedFunc[Counter](ore.Scoped, func(ctx context.Context) (Counter, context.Context) {
    return &simpleCounter{}, ctx
}, "simple")

//atomicCounter
ore.RegisterKeyedFunc[Counter](ore.Scoped, func(ctx context.Context) (Counter, context.Context) {
    return &atomicCounter{}, ctx
}, "exotic")

//rgbCounter
ore.RegisterKeyedCreator[Counter](ore.Singleton, &rgbCounter{}, "exotic")

//quantumCounter
ore.RegisterKeyedFunc[Counter](ore.Transient, func(ctx context.Context) (Counter, context.Context) {
    return &quantumCounter{}, ctx
}, "exotic")
```

let's now resolve a list of implementation of the service by key:

```go
exoticCounters, ctx := ore.GetKeyedList[Counter](context.Background(), "exotic")
fmt.Println(len(exoticCounters)) // outputs: 3

nonKeyedCounters, ctx := ore.GetList[Counter](context.Background())
fmt.Println(len(nonKeyedCounters)) // outputs: 0

simpleCounters, ctx := ore.GetKeyedList[Counter](context.Background(), "simple")
fmt.Println(len(simpleCounters)) // outputs: 1
```

let's see what happens when we try to resolve a single implementation:

```go
exoticCounter, ctx := ore.GetKeyed[Counter](context.Background(), "exotic")
// `exoticCounter` is of type `quantumCounter{}`

simpleCounter, ctx := ore.GetKeyed[Counter](context.Background(), "simple")
// `simpleCounter` is of type `simpleCounter{}`


counter, ctx := ore.Get[Counter](context.Background())
// this will panic, because there is no non-keyed implementation for `Counter`


awesomeCounter, ctx := ore.GetKeyed[Counter](context.Background(), "awesome")
// this will panic, because there is no implementation for `Counter` with key `awesome`
```