---
sidebar_position: 2
---


# Keyed Implementations

In **Ore**, you may need to register multiple implementations of the same interface when different versions or behaviors are required for distinct scenarios. This is accomplished using **keyed services**, where each implementation is identified by a unique key.

### Why Register Multiple Implementations?

1. **Scenario-Specific Behavior**:  
   You might need different implementations of an interface for testing, production, or specialized business logic.

2. **Modularity**:  
   Different modules of your application may rely on distinct implementations of the same interface.

3. **Decoupling**:  
   Separating interface definitions from their implementations allows you to switch or extend functionality with minimal changes.

4. **Dynamic Selection**:  
   Use cases where the implementation must be chosen dynamically at runtime based on a condition.

---

### How to Register Multiple Implementations in Ore

Here's an example demonstrating how to register and resolve multiple implementations of an interface using keys:

#### Code Example:

```go
// Define an interface
type Greeter interface {
	Greet() string
}

// Implementation 1: Friendly Greeter
type FriendlyGreeter struct{}

func (g *FriendlyGreeter) Greet() string {
	return "Hello! Nice to meet you!"
}

// Implementation 2: Formal Greeter
type FormalGreeter struct{}

func (g *FormalGreeter) Greet() string {
	return "Good day. How may I assist you?"
}
```

```go
// Register multiple implementations with unique keys
ore.RegisterKeyedFunc[Greeter](ore.Scoped, func(ctx context.Context) (Greeter, context.Context) {
    return &FriendlyGreeter{}, ctx
}, "friendly")

ore.RegisterKeyedFunc(ore.Transient, func(ctx context.Context) (Greeter, context.Context) {
    return &FormalGreeter{}, ctx
}, "formal")


ctx := context.Background()

// Resolve based on keys
friendlyGreeter, ctx := ore.GetKeyed[Greeter](ctx, "friendly") 
fmt.Println(friendlyGreeter.Greet()) // Output: Hello! Nice to meet you!


formalGreeter, ctx := ore.GetKeyed[Greeter](ctx, "formal")
fmt.Println(formalGreeter.Greet())  // Output: Good day. How may I assist you?
```


---

### Key Benefits of Keyed Services in Ore

1. **Clarity**: Each implementation is clearly identified, making the code easier to understand and maintain.
2. **Flexibility**: You can dynamically decide which implementation to use based on runtime conditions.
3. **Scalability**: Adding new implementations requires minimal changes to existing code.

This approach ensures your application is modular, flexible, and prepared for complex dependency scenarios.