---
sidebar_position: 5
---

# Graceful Termination

### Ore's Graceful Termination Features

Ore provides built-in tools to manage **graceful application termination** and **graceful context termination**,
ensuring proper resource cleanup for services with Singleton or Scoped lifetimes. These features automate the orderly
disposal of resources, allowing for clean and predictable shutdown processes.

---

### Graceful Application Termination

On application termination, it is essential to clean up resources used by **Singleton** services, such as closing
database connections, stopping background tasks, or flushing logs. Ore helps by identifying and invoking cleanup methods
on only the **created Singleton instances**.

#### Key Features:

1. **Interface-Based Cleanup**:
    - Services implementing a specific cleanup interface (e.g., `Shutdowner`) are automatically identified.
    - Uncreated (lazily registered) instances are ignored.

2. **Dependency-Aware Order**:
    - Services are shut down in reverse order of their resolution, ensuring dependencies are cleaned up first.
    - Example: If `A` depends on `B` and `C`, `B` and `C` are shut down before `A`.

3. **Inclusive of Keyed Services**:
    - Includes both regular and keyed Singleton instances.

---

#### Code Example:

```go
// Define a shutdown interface
type Shutdowner interface {
  Shutdown()
}

// Register Singleton services
ore.RegisterSingleton[Logger](&Logger{})               // *Logger implements Shutdowner
ore.RegisterSingleton[DBRepository](&SomeRepository{})       // *SomeRepository implements Shutdowner
ore.RegisterKeyedSingleton[HealthCheckService](&SomeService{}, "module") // *SomeService implements Shutdowner

// On application termination
shutdowables := ore.GetResolvedSingletons[Shutdowner]() // Retrieve all resolved instances of Shutdowner

// Shutdown all services
for _, instance := range shutdowables {
   instance.Shutdown()
}
```

**Key Notes**:

- Only Singleton instances that were resolved during the application lifetime are included.
- Instances are returned in reverse resolution order, ensuring proper cleanup.

---

### Graceful Context Termination

For services with **Scoped lifetimes**, cleanup should occur when their associated context is terminated. Ore helps
manage this through the `Dispose()` method, invoked on all Scoped instances implementing a specific interface (e.g.,
`Disposer`).

#### Key Features:

1. **Context-Specific Cleanup**:
    - Only services resolved during the context lifetime are included in the cleanup process.

2. **Dependency-Aware Order**:
    - Services are disposed of in reverse resolution order, ensuring dependencies are cleaned up first.

3. **Inclusive of Keyed Services**:
    - Includes both regular and keyed Scoped instances.

---

#### Code Example:

```go
// Define a disposer interface
type Disposer interface {
  Dispose()
}

// Register Scoped services
ore.RegisterCreator[EmailService](ore.Scoped, &SomeDisposableService{}) // *SomeDisposableService implements Disposer

// Simulate a new request with its own context
ctx, cancel := context.WithCancel(context.Background())

// Start a goroutine to handle cleanup when the context is canceled
go func() {
  <-ctx.Done() // Wait for context cancellation
  disposables := ore.GetResolvedScopedInstances[Disposer](ctx) // Retrieve all resolved Scoped Disposer instances
  for _, d := range disposables {
    d.Dispose() // Clean up resources
  }
}()

// Resolve a Scoped service
ore.Get[*SomeDisposableService](ctx)

// Simulate context cancellation
cancel() // Triggers cleanup
```

**Key Notes**:

- Only Scoped instances resolved within the specific context are included.
- Instances are returned in reverse resolution order.

---

### Summary

- **Application Termination**: Use `ore.GetResolvedSingletons[TInterface]()` to retrieve and clean up Singleton
  instances during application shutdown.
- **Context Termination**: Use `ore.GetResolvedScopedInstances[TInterface](context)` to clean up Scoped instances during
  context cancellation.
- **Dependency-Aware Cleanup**: Ore ensures that dependencies are shut down or disposed of before their dependents,
  avoiding potential resource leaks or deadlocks.

These features simplify resource management and make it easy to implement predictable and clean shutdown processes in
applications using Ore.