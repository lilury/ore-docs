---
sidebar_position: 7
---

# Isolated Containers

### Ore's Isolated Containers (a.k.a Modules)

Ore supports the creation of **isolated containers**, enabling advanced use cases such as **Modular Monolith
Architecture**, where each module maintains its own independent dependency container. This feature ensures clean
boundaries between modules, helping you enforce separation of concerns and avoid accidental cross-module dependencies.

---

### Key Features of Isolated Containers

1. **Independent Dependency Graphs**:  
   Each container has its own registration and resolution process, isolating modules from each other.

2. **Default and Custom Containers**:
    - The **Default Container** is sufficient for most use cases.
    - Custom containers can be created for modular architectures or advanced scenarios.

3. **Minimal Overlap**:  
   Containers are completely independent. To enforce module isolation, avoid cross-container access.

---

### Use Cases

- **Modular Monolith Architecture**:  
  Separate containers for each module, such as `Broker` and `Trader`, allow individual dependency graphs and better
  encapsulation.

- **Testing and Isolation**:  
  Custom containers are useful in unit tests to isolate dependencies and mock certain services.

---

### How It Works

1. **Creating a New Container**:  
   Use `ore.NewContainer()` to create a custom container.

2. **Registering Services to a Container**:  
   Register services using functions like `RegisterFuncToContainer`, `RegisterSingletonToContainer`, or
   `RegisterPlaceholderToContainer`.

3. **Resolving Dependencies from a Container**:  
   Use functions like `GetFromContainer` or `GetListFromContainer` to resolve dependencies.

4. **Validation and Sealing**:
    - **Seal** the container to prevent further registrations using `container.Seal()`.
    - **Validate** the dependency graph with `container.Validate()`.
    - Disable runtime validation (if needed) with `container.DisableValidation = true`.

---

### Example: Modular Monolith with Multiple Containers

#### Broker Module

```go
// Create a container for the Broker module
brokerContainer := ore.NewContainer()

// Register a Broker service
ore.RegisterFuncToContainer(brokerContainer, ore.Singleton, func(ctx context.Context) (*Broker, context.Context) {
  brs, ctx := ore.GetFromContainer[*BrokerageSystem](brokerContainer, ctx) // Resolve dependencies from the Broker container
  return &Broker{brs}, ctx
})

// Resolve the Broker service
broker, _ := ore.GetFromContainer[*Broker](brokerContainer, context.Background())
```

#### Trader Module

```go
// Create a container for the Trader module
traderContainer := ore.NewContainer()

// Register a Trader service
ore.RegisterFuncToContainer(traderContainer, ore.Singleton, func(ctx context.Context) (*Trader, context.Context) {
  mkp, ctx := ore.GetFromContainer[*MarketPlace](traderContainer, ctx) // Resolve dependencies from the Trader container
  return &Trader{mkp}, ctx
})

// Resolve the Trader service
trader, _ := ore.GetFromContainer[*Trader](traderContainer, context.Background())
```

---

### Best Practices

1. **Module Isolation**:
    - Ensure each module interacts only with its own container.
    - Avoid exposing internal container references to other modules.

2. **Dependency Validation**:
    - Use `container.Validate()` to check for circular dependencies and other issues.
    - Seal containers after registration to prevent accidental modifications.

3. **Cross-Module Interaction**:
    - If interaction between modules is required, expose only the required interfaces or services, not the entire
      container.

---

### Comparison: Default vs Custom Containers

| Default Container     | Custom Container                   |
|-----------------------|------------------------------------|
| Get                   | GetFromContainer                   |
| GetList               | GetListFromContainer               |
| GetResolvedSingletons | GetResolvedSingletonsFromContainer |
| RegisterAlias         | RegisterAliasToContainer           |
| RegisterSingleton     | RegisterSingletonToContainer       |
| RegisterCreator       | RegisterCreatorToContainer         |
| RegisterFunc          | RegisterFuncToContainer            |
| RegisterPlaceholder   | RegisterPlaceholderToContainer     |
| ProvideScopedValue    | ProvideScopedValueToContainer      |

---

### Summary

Ore's **Isolated Containers** provide a robust mechanism for creating modular applications with strict boundaries
between components. By leveraging this feature, you can:

- Maintain clean dependency graphs per module.
- Avoid unintended coupling between modules.
- Build scalable, maintainable applications with clear separation of concerns.  