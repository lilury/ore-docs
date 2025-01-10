---
sidebar_position: 8
---

# Benchmarks

```bash
goos: windows
goarch: amd64
pkg: github.com/firasdarwish/ore
cpu: 13th Gen Intel(R) Core(TM) i9-13900H
BenchmarkRegisterFunc-20                 5612482               214.6 ns/op
BenchmarkRegisterCreator-20              6498038               174.1 ns/op
BenchmarkRegisterSingleton-20            5474991               259.1 ns/op
BenchmarkInitialGet-20                   2297595               514.3 ns/op
BenchmarkGet-20                          9389530               122.1 ns/op
BenchmarkInitialGetList-20               1000000               1072 ns/op
BenchmarkGetList-20                      3970850               301.7 ns/op
PASS
ok      github.com/firasdarwish/ore     10.883s
```

Checkout also [examples/benchperf/README.md](https://github.com/firasdarwish/ore/blob/main/examples/benchperf/README.md)