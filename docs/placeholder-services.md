---
sidebar_position: 6
---

# Placeholder Services

### Ore's Placeholder Service Registration

Ore's **Placeholder Service Registration** is designed to handle scenarios where certain dependencies cannot be provided
at the time of service registration but become available later during runtime. This feature enables you to build a
flexible and dynamic dependency graph that adapts to runtime conditions.

---

### Key Features of Placeholder Registration

1. **Deferred Dependency Injection**:
    - Allows you to define services with dependencies that are resolved only when specific runtime events or requests
      occur.

2. **Scoped Placeholder Values**:
    - Values can be dynamically provided to placeholders during the handling of specific requests or contexts.

3. **Seamless Integration**:
    - Dependencies can be resolved as part of the normal dependency injection workflow once the placeholder is
      populated.

4. **Failsafe Mechanism**:
    - Resolving services dependent on unfulfilled placeholders triggers a panic, helping catch missing runtime
      dependencies early.

---

### How It Works

1. **Registering a Placeholder**:  
   Use `ore.RegisterKeyedPlaceholder[T](key...)` to register a placeholder for a future dependency value.
    - The placeholder acts as a promise for a value that will be provided later.

2. **Providing a Runtime Value**:  
   Use `ore.ProvideKeyedScopedValue(ctx, value, key...)` to inject a value into the placeholder dynamically based on
   runtime conditions or events.
    - The injected value is associated with the current context and used to resolve dependencies.

3. **Resolution Behavior**:
    - Dependencies relying on placeholders resolve successfully once the value is provided.
    - Real resolvers (e.g., registered functions or creators) take precedence over placeholders if both are available.

---

### Example: Injecting Runtime Values

#### Scenario:

You have a service `SomeService` that depends on a configuration value `someConfig`. The value of `someConfig` is
determined by the user role, which is only available at runtime.

#### Code Example:

```go
// Define a service
type SomeService struct {
  Config string
}

// Register a service that depends on "someConfig"
ore.RegisterFunc[*SomeService](ore.Scoped, func(ctx context.Context) (*SomeService, context.Context) {
  someConfig, ctx := ore.GetKeyed[string](ctx, "someConfig") // Retrieve the runtime value
  return &SomeService{Config: someConfig}, ctx
})

// Register a placeholder for "someConfig"
ore.RegisterKeyedPlaceholder[string]("someConfig")

// Simulate a new request
ctx := context.Background()
// Example: User role determined by the request
ctx = context.WithValue(ctx, "role", "admin")

// Dynamically inject a value into the placeholder based on the user's role
userRole := ctx.Value("role").(string)
if userRole == "admin" {
  ctx = ore.ProvideKeyedScopedValue(ctx, "Admin config", "someConfig")
} else if userRole == "supervisor" {
  ctx = ore.ProvideKeyedScopedValue(ctx, "Supervisor config", "someConfig")
} else {
  ctx = ore.ProvideKeyedScopedValue(ctx, "Default config", "someConfig")
}

// Resolve the service with the dynamically provided configuration
service, ctx := ore.Get[*SomeService](ctx)
fmt.Println(service.Config) // Output: "Admin config" (based on the role)
```

---

### Summary of Key Functions

1. **`ore.RegisterKeyedPlaceholder[T](key...)`**
    - Registers a placeholder for a dependency that will be provided later.
    - Supports Scoped lifetime.

2. **`ore.ProvideKeyedScopedValue(ctx, value T, key...)`**
    - Injects a runtime value into a registered placeholder.
    - The value is specific to the given context.

3. **Resolution Precedence**:
    - Real resolvers (e.g., functions or creators) take precedence over placeholder-provided values.

---

### Benefits of Placeholder Registration

1. **Dynamic Configuration**:
    - Tailor services dynamically based on user input, request context, or runtime conditions.

2. **Improved Modularity**:
    - Decouple service definitions from runtime logic.

3. **Error Prevention**:
    - Avoid unintentional dependency issues with built-in validation for unfulfilled placeholders.

With Ore's placeholder service registration, you gain the flexibility to manage dependencies dynamically, ensuring that
your applications remain responsive to runtime needs.