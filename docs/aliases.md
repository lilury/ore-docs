---
sidebar_position: 4
---

# Aliases

The **Aliases** feature in Ore allows developers to associate multiple implementations of a type (e.g., structs) with a
common interface. This provides flexibility when resolving dependencies and simplifies service management.

Aliases enable dynamic linking between interfaces and concrete implementations, helping you write cleaner and more
modular code.

---

### Key Benefits of Aliases

1. **Flexible Resolution**:
    - Allows interfaces to be dynamically linked to one or more implementations.
    - Enables resolving dependencies without explicitly registering the interface itself.

2. **Precedence Handling**:
    - When multiple implementations are linked to the same alias, the most recently registered implementation takes
      precedence.

3. **Dynamic Scoping**:
    - Supports scoped resolution, where aliases are resolved based on a specific key.

4. **Collection Support**:
    - Easily retrieve all implementations associated with an alias as a list for batch processing or workflows.

---

### How Aliases Work

- **Linking Aliases**:  
  An alias is created by linking an interface to one or more concrete implementations using `RegisterAlias`.

- **Resolution Precedence**:
    - If multiple implementations are registered, the last registered alias takes precedence when using `Get`.
    - Collections (`GetList`) return all linked implementations.

- **Key-Based Scoping**:  
  Aliases can be scoped by a **key**. When resolving with a key, only services registered under that key are considered.

---

### Example: Using Aliases in Ore

#### Code Example

```go
// Define an interface
type IPerson interface{}

// Define implementations
type Broker struct {
	Name string
} // implements IPerson

type Trader struct {
	Name string
} // implements IPerson
```

Now let's register the `struct`s and link them to `IPerson`:

```go
// Register services
ore.RegisterFunc(ore.Scoped, func(ctx context.Context) (*Broker, context.Context) {
    return &Broker{Name: "Peter"}, ctx
})

ore.RegisterFunc(ore.Scoped, func(ctx context.Context) (*Broker, context.Context) {
    return &Broker{Name: "John"}, ctx
})

ore.RegisterFunc(ore.Scoped, func(ctx context.Context) (*Trader, context.Context) {
    return &Trader{Name: "Mary"}, ctx
})

// Register aliases
ore.RegisterAlias[IPerson, *Trader]() // Link IPerson to *Trader
ore.RegisterAlias[IPerson, *Broker]() // Link IPerson to *Broker
```

Now let's resolve the alias:

```go
// Resolve the alias
person, _ := ore.Get[IPerson](context.Background()) // Returns the last registered broker (John)
fmt.Printf("Resolved Person: %v\n", person)

// Get a list of all registered implementations
personList, _ := ore.GetList[IPerson](context.Background())
fmt.Println("All Persons:")
for _, p := range personList {
    fmt.Printf(" - %v\n", p)
}
```

NOTE: Precedence here is based on the most recently **LINKED** object, not the most recently *registered*.

---

### Key-Based Aliases

You can also use **key-based scoping** to resolve specific implementations associated with a key:

#### Key-Based Example:

```go
ore.RegisterKeyedFunc(ore.Scoped, func(ctx context.Context) (*Broker, context.Context) {
    return &Broker{Name: "Keyed John"}, ctx
}, "module1")

person, _ := ore.GetKeyed[IPerson](context.Background(), "module1") // Resolves Broker registered under "module1"
fmt.Printf("Resolved Person with key: %v\n", person)
```

---

### When to Use Aliases

- When you want to group and dynamically resolve services based on runtime conditions.

Ore’s aliases provide powerful tools for managing dependencies flexibly and efficiently, especially in complex or
modular systems.