---
sidebar_position: 1
---

# Singleton

### What Are Singletons?

A **singleton** is a design pattern that ensures an object has only one instance throughout the application’s lifecycle. It provides a global point of access to this instance, making it ideal for managing shared resources like configuration, logging, or database connections.

### Eager Singletons vs. Lazy Singletons

1. **Eager Singletons**:
    - The instance is created immediately when the application starts or when the singleton is registered.
    - Pros:
        - Guarantees the instance is ready to use when needed.
        - Can help detect initialization issues early in the application lifecycle.
    - Cons:
        - Increases startup time.
        - Wastes resources if the singleton is never used.

2. **Lazy Singletons**:
    - The instance is created only when it is first accessed.
    - Pros:
        - Optimizes resource usage by delaying initialization until necessary.
        - Reduces startup overhead.
    - Cons:
        - Initialization errors occur at runtime, potentially causing delays or failures during execution.

Both types are useful depending on the context—eager singletons for critical, always-needed services, and lazy singletons for optional or resource-heavy components.

<br />

## Sample Service

```go
type MyService interface {
    SayHello()
}

type myServiceImpl struct{}

func (s *myServiceImpl) SayHello() {
    fmt.Println("Hello from MyService!")
}
```

<br/>

## Eager Singleton

```go
// initialize the service
myService := &myServiceImpl{}

// registering `myService` as an eager singleton
ore.RegisterSingleton[MyService](myService)
```

```go
// resolving the eager singleton service `MyService`
myService, ctx := ore.Get[MyService](context.Background())
```

<br/>

## Lazy Singleton

When registering a service as lazy singleton (or scoped, transient), Ore will initialize the service
later when it's needed. So Ore needs a way to construct the service.
There is two ways to tell Ore how to construct a service:
- Using `Creator[T]` interface
- Using anonymous functions

we will explore later on how to use both methods, but for now we'll use an anonymous function

```go
// registering `myService` as a lazy singleton
ore.RegisterFunc[MyService](ore.Singleton, func(ctx context.Context) (MyService, context.Context) {
    return &myServiceImpl{}, ctx
})
```

as you see, we called `ore.RegisterFunc[MyService]`,
- first, we've passed the lifetime we want, which is `ore.Singleton`
- second, we've declared an anonymous function which takes 1 argument `(ctx context.Context)`
and it returns 2 values `(MyService, context.Context)`
- inside the anonymous func body, we've constructed a new instance of `myServiceImpl` which implements `MyService`, and returned it alongside the `ctx`.


Now, Ore knows exactly how to construct a singleton instance of `MyService` when you need it:
```go
// resolving the lazy singleton service `MyService`
myService, ctx := ore.Get[MyService](context.Background())
```

On the first call to `ore.Get[MyService](ctx)`, Ore will make a new instance of `MyService` and return it,
and it will return the same instance of every subsequent call to `ore.Get[MyService](ctx)`.
