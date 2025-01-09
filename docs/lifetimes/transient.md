---
sidebar_position: 3
---

# Transient

### What Are Transient Services?

**Transient services** are services that are created each time they are requested. They are not cached or reused between
requests. In dependency injection, transient services are typically used for stateless objects that don't need to retain
data between instances. Every time a transient service is requested, a new instance is created, ensuring a fresh and
independent state for each usage. This is ideal for scenarios where services don’t need to maintain state or share
resources between invocations, improving isolation and avoiding unintended side effects from shared state.

## Sample Service

```go
type Counter interface {
    GetCount() int
    AddOne()
}

type simpleCounterImpl struct {
    counter int
}

func (c *simpleCounterImpl) AddOne() {
    c.counter += 1
}

func (c *simpleCounterImpl) GetCount() int {
    return c.counter
}
```

## Registering Transient Services

```go
// registering `simpleCounterImpl` as a transient service
ore.RegisterFunc[Counter](ore.Transient, func(ctx context.Context) (Counter, context.Context) {
    return &simpleCounterImpl{
        counter: 0,
    }, ctx
})
```

Notice the first argument is `ore.Transient`.

## Resolving Transient Service

```go
// resolving the transient service `Counter`
ctx := context.Background()
counter, ctx := ore.Get[Counter](ctx)
fmt.Println(counter.GetCount()) // returns 0

counter.AddOne()

fmt.Println(counter.GetCount()) // returns 1

// resolving `Counter` again using the same scope (ctx)
counter, ctx = ore.Get[Counter](ctx)
counter.AddOne()
fmt.Println(counter.GetCount()) // returns 1


// resolving `Counter` again using a *NEW* scope
counter, ctx = ore.Get[Counter](context.Background())
counter.AddOne()
fmt.Println(counter.GetCount()) // returns 1
```

Note that on each time `ore.Get[Counter](ctx)` is called, a fresh, new instance of `Counter` is being initialized and
returned no matter the scope (context).