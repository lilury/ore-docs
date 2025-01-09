---
sidebar_position: 2
---

# Scoped

### What Are Scoped Services?

**Scoped services** are a type of dependency in application development whose lifecycle is tied to a specific **scope**.
A scope typically refers to a well-defined context, such as a single web request, a database transaction, or any other
logical unit of work.

### Key Features of Scoped Services:

- **Instance Per Scope**: A new instance of the service is created for each scope, and the same instance is reused
  throughout that scope.
- **Isolation**: Scoped services allow separation of resources, ensuring that state or data within one scope does not
  leak into another.
- **Efficient Resource Management**: By limiting the lifetime of the service to a scope, unnecessary memory or resource
  usage can be minimized.

### Common Use Cases:

1. **Web Applications**: Scoped services are commonly used in web frameworks where a single instance is shared for the
   duration of an HTTP request. This ensures data consistency and simplifies state management.
2. **Transactional Systems**: In database applications, scoped services can ensure that dependencies (like transaction
   contexts) are confined to a single transaction's lifespan.

Scoped services offer a balance between performance and resource isolation, making them ideal for managing dependencies
in dynamic, context-sensitive workflows.

## Scopes by context.Context

Using `context.Context` for scoped services in **Ore** aligns with Go's standard practice for managing request-scoped
data, cancellations, and deadlines. It ensures services are tied to specific contexts (like requests), preventing
leakage across scopes. When a request is canceled or times out, the associated services cannot be reused and will be disposed by the GC,
optimizing resource management. The context also allows for easy propagation of service data without needing explicit
dependency injection, keeping the code clean and maintainable. Additionally, Go’s concurrency model benefits from
context, enabling efficient management of scoped services across concurrent goroutines. This design choice embraces Go's
principles of simplicity and composability.

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


## Registering Scoped Services

As mentioned earlier in `Singletons`, there are 2 ways to register scoped service, for now we'll use `anonymous func`:

```go
// registering `simpleCounterImpl` as a scoped service
ore.RegisterFunc[Counter](ore.Scoped, func(ctx context.Context) (Counter, context.Context) {
    return &simpleCounterImpl{
        counter: 0,
    }, ctx
})
```

Notice the first argument is `ore.Scoped`.


## Resolving Scoped Service

```go
// resolving the scoped service `Counter`
ctx := context.Background()
counter, ctx := ore.Get[Counter](ctx)
fmt.Println(counter.GetCount()) // returns 0

counter.AddOne() 

fmt.Println(counter.GetCount()) // returns 1

// resolving `Counter` again using the same scope (ctx)
counter, ctx = ore.Get[Counter](ctx)
counter.AddOne()
fmt.Println(counter.GetCount()) // returns 2


// resolving `Counter` again using a *NEW* scope
counter, ctx = ore.Get[Counter](context.Background())
counter.AddOne()
fmt.Println(counter.GetCount()) // returns 1
```