---
sidebar_position: 3
---

# Validation

### Introduction to Ore's Registration Validation

Ore includes built-in **registration validation** to ensure the reliability and correctness of your dependency injection
setup. This feature helps detect and prevent common issues during application startup, making debugging and maintenance
easier.

---

### Key Validation Checks

1. **Missing Dependency**:  
   Occurs when a required service or dependency is not registered in the container.
    - **Example**: A service attempts to resolve a dependency that has no corresponding resolver.
    - **Impact**: Leads to runtime errors when the application tries to access the missing service.

2. **Circular Dependency**:  
   Happens when two or more dependencies depend on each other, creating a loop.
    - **Example**: Service A depends on Service B, which in turn depends on Service A.
    - **Impact**: Causes stack overflows or hangs during resolution.

3. **Lifetime Misalignment**:  
   Occurs when a service with a **longer lifetime** (e.g., Singleton) depends on a service with a **shorter lifetime** (
   e.g., Transient).
    - **Example**: A Scoped service resolves a Transient dependency, causing the Scoped service to potentially hold
      outdated references.
    - **Impact**: Leads to inconsistent or unexpected behavior.

---

### Why Registration Validation Matters

- **Prevents Runtime Failures**: Catch issues at startup rather than during execution.
- **Improves Maintainability**: Makes it easier to manage complex dependencies in large applications.
- **Enhances Debugging**: Clear error messages point directly to problematic registrations.
- **Promotes Best Practices**: Encourages clean, modular, and well-structured dependency design.

---

### Example: Enabling Validation in Ore

By default, Ore performs validation automatically during each to call `ore.Get` or `ore.GetList` and their keyed
variants.

You could also manually trigger the validation check by calling `ore.Validate()`.

## Performance

As mentioned earlier, Ore validates the dependency chain each time it is resolving a service.
This may cause some performance issues in some rare cases, so the wise thing to do would be to disable this behavior:

```go
ore.DisableValidation = true
```

## Recommendations

We highly recommend to seal the container after you finish registering all your service:

```go
ore.Seal()
```

`ore.Seal()` This will cause Ore to disallow (it'll panic) registration of any new services to the container.

After that, it is very helpful to call `ore.Validate()` once at the application starting.

NOTE: `ore.Validate()` will try to resolve all registered services and their dependencies in order to make sure
everything is working correctly, and then will clear them up.

NOTE: Services constructors logic should be very simple, straight-forward and always outputs the same result.

And after all that being said, you may disable Ore's automatic validation if needed.


---

### Best Practices for Avoiding Validation Issues

1. **Plan Dependencies**: Map out dependencies before implementation to identify potential issues early.
2. **Use Lazy Initialization**: For circular dependencies, consider resolving some dependencies lazily.
3. **Align Lifetimes**: Ensure services with shorter lifetimes don’t leak into longer-living ones.
4. **Validate Early**: Use automated tests to validate container configurations regularly.

Ore's validation tools make it easier to create reliable and maintainable dependency graphs, empowering you to focus on
building robust applications.