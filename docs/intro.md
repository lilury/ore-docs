---
sidebar_position: 1
---


# Introduction

**Ore** is a lightweight, generic, and simple dependency injection (DI) container for Go, inspired by ASP.NET's DI principles. It facilitates object lifetime management and inversion of control in Go applications, offering features such as:

- **Singletons**: Ensuring a single instance throughout the application.
- **Transients**: Creating a new instance with each request.
- **Scoped Instances**: Tying instances to specific contexts or scopes.
- **Lazy Initialization**: Efficient resource utilization by delaying object creation until necessary.
- **Multiple Implementations**: Managing several implementations of the same interface.
- **Keyed Services Injection**: Differentiating between multiple implementations using keys.
- **Concurrency Safety**: Utilizing mutexes to ensure safe concurrent access.
- **Placeholder Service Registration**: Simplifying scenarios where dependencies are resolved at runtime.
- **Isolated Containers**: Supporting multiple isolated, modular containers for scalable architectures.
- **Aliases**: Registering type mappings to treat one type as another.
- **Runtime Validation**: Enhancing error handling during startup or testing phases.
- **Graceful Termination**: Ensuring proper cleanup of resources during application shutdown.

## Installation

To install Ore, use the following command:

```bash
go get -u github.com/firasdarwish/ore
```

## Getting Started

Import the package in your Go application:

```go
import "github.com/firasdarwish/ore"
```

Here's a simple example of how to register and resolve a singleton service:

```go
type MyService interface {
    SayHello()
}

type myServiceImpl struct{}

func (s *myServiceImpl) SayHello() {
    fmt.Println("Hello from MyService!")
}

// register an eager singleton
ore.RegisterSingleton[MyService](&myServiceImpl{})

// resolve the service
ctx := context.Background()
myService, ctx := ore.Get[MyService](ctx)

myService.SayHello()
```


## License

Ore is licensed under the MIT License. See the [LICENSE](https://github.com/firasdarwish/ore/blob/main/LICENSE) file for more details.

## Contributing

Contributions are welcome! Feel free to open issues or submit pull requests on the [GitHub repository](https://github.com/firasdarwish/ore).