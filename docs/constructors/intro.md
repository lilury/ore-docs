---
sidebar_position: 1
---

# Intro

Dependency Injection (DI) containers need a way to construct objects because their primary purpose is to manage and
resolve dependencies for the objects in an application. Many objects depend on other objects or services, and these
dependencies often need to be instantiated, configured, and injected at runtime. Here's why construction mechanisms are
essential:

1. **Automatic Dependency Resolution**: DI containers resolve dependencies by analyzing the required dependencies of an
   object and providing the appropriate instances. To do this, the container must construct the objects as needed,
   ensuring all required dependencies are fulfilled.

2. **Abstraction of Object Creation**: Instead of hardcoding object creation in application code, DI containers
   centralize this responsibility. This allows for more flexible and modular design by decoupling the object's creation
   from its usage.

3. **Lifecycle Management**: DI containers manage the lifecycle of objects (e.g., singleton, transient, scoped).
   Constructing objects according to these lifecycles requires the container to have precise control over when and how
   objects are instantiated.

4. **Complex Object Graphs**: In real-world applications, objects often have deep and complex dependency trees. DI
   containers can recursively construct and resolve these dependencies, simplifying application code and ensuring
   consistency.

Without the ability to construct objects, DI containers would lose their primary utility of automating and simplifying
dependency management.

Ore provides 2 ways to tell it how to construct objects:
- Anonymous Functions
- `Creator[T]` interface