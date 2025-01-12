---
sidebar_position: 5
---

# Graceful Termination
 

### Graceful Termination Overview

Ore provides tools to assist with **graceful application termination** and **graceful context termination**. It enables
you to retrieve resolved instances of services, allowing you to manage resource cleanup based on your application's
requirements. Ore integrates seamlessly with existing projects, supporting any custom cleanup interfaces you define.

---  

### Graceful Application Termination

During application termination, it is crucial to clean up resources used by Singleton services, such as closing database
connections, stopping background tasks, or flushing logs. Ore simplifies this process by helping you retrieve resolved
Singleton instances that implement specific interfaces.

#### Key Features:

1. **Custom Cleanup Interfaces**:
    - Ore works with any interface you define for resource management, such as `Shutdowner`, `Closer`, or `Flusher`.
    - These interfaces are part of **your application**, not Ore.

2. **Dependency-Aware Order**:
    - Singleton services are retrieved in reverse resolution order, ensuring that dependencies are addressed before
      their dependents.

3. **Selective Retrieval**:
    - Ore only retrieves Singleton instances that were resolved during the application's lifecycle. Unresolved
      Singletons are excluded.

---  

#### Example: Retrieving Resolved Singleton Instances

```go
// Example cleanup interfaces defined in your application
type Shutdowner interface {
  Shutdown()
}
type Closer interface {
  Close()
}

// Register Singleton services
ore.RegisterSingleton[Logger](&Logger{})               // *Logger implements Shutdowner
ore.RegisterSingleton[DBRepository](&SomeRepository{}) // *SomeRepository implements Closer

// On application termination, retrieve resolved Singleton instances
shutdownables := ore.GetResolvedSingletons[Shutdowner]()
closeables := ore.GetResolvedSingletons[Closer]()

// Invoke cleanup methods
for _, o := range shutdownables {
  o.Shutdown()
}
for _, o := range closeables {
  o.Close()
}
```  

**Note**: Ore only retrieves instances that have already been resolved. The actual cleanup logic (e.g., `Shutdown` or
`Close` methods) is entirely up to your application.

---  

### Graceful Context Termination

For Scoped services, Ore helps manage cleanup when their associated context ends. It does so by allowing you to retrieve
Scoped instances that match specific interfaces within the given context.

#### Key Features:

1. **Custom Cleanup Interfaces**:
    - Scoped cleanup works with any interface (e.g., `Disposer`, `Closer`, `Flusher`) defined in your application.
    - Ore provides the means to retrieve Scoped instances implementing these interfaces.

2. **Dependency-Aware Order**:
    - Scoped instances are retrieved in reverse resolution order to ensure dependencies are addressed before dependents.

3. **Context-Specific Retrieval**:
    - Ore retrieves only Scoped instances that were resolved within the specific context, avoiding interference with
      other contexts.

---  

#### Example: Retrieving Scoped Instances

```go
// Example cleanup interfaces defined in your application
type Disposer interface {
  Dispose()
}
type Flusher interface {
  Flush()
}

// Register Scoped services
ore.RegisterCreator[EmailService](ore.Scoped, &EmailService{}) // Implements Disposer
ore.RegisterCreator[CacheService](ore.Scoped, &CacheService{}) // Implements Flusher

// Simulate a context
ctx, cancel := context.WithCancel(context.Background())

// Cleanup on context termination
go func() {
  <-ctx.Done() // Wait for context cancellation
  disposables := ore.GetResolvedScopedInstances[Disposer](ctx)
  flushables := ore.GetResolvedScopedInstances[Flusher](ctx)

  // Invoke cleanup methods
  for _, d := range disposables {
    d.Dispose()
  }
  for _, f := range flushables {
    f.Flush()
  }
}()

// Resolve services in the context
emailService, ctx := ore.Get[EmailService](ctx)
cacheService, ctx := ore.Get[CacheService](ctx)

// Simulate context cancellation
cancel() // Triggers cleanup
```  

**Note**: Ore does not perform the cleanup itself. It only retrieves resolved instances, leaving the cleanup logic to
your application.

---  

### Summary

- Ore provides tools to retrieve resolved Singleton or Scoped instances using any user-defined interface.
- **Application Termination**: Use `ore.GetResolvedSingletons` to retrieve resolved Singleton instances.
- **Context Termination**: Use `ore.GetResolvedScopedInstances` to retrieve Scoped instances within a given context.
- **Customizable Cleanup**: Ore works with any interface you define, ensuring seamless integration with existing
  projects.