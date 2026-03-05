---
sidebar_position: 8
---

# Benchmarks

```bash
goos: windows
goarch: amd64
pkg: github.com/firasdarwish/ore
cpu: 13th Gen Intel(R) Core(TM) i9-13900H
BenchmarkRegisterFunc
BenchmarkRegisterFunc-20         	 5927286	       197.1 ns/op
BenchmarkRegisterCreator
BenchmarkRegisterCreator-20      	 6831612	       181.5 ns/op
BenchmarkRegisterSingleton
BenchmarkRegisterSingleton-20    	 5541734	       200.1 ns/op
BenchmarkInitialGet
BenchmarkInitialGet-20           	 2704546	       430.0 ns/op
BenchmarkGet
BenchmarkGet-20                  	10122558	       109.0 ns/op
BenchmarkInitialGetList
BenchmarkInitialGetList-20       	 1327143	       894.4 ns/op
BenchmarkGetList
BenchmarkGetList-20              	 4105911	       285.8 ns/op
PASS
```

Checkout also [examples/benchperf/README.md](https://github.com/firasdarwish/ore/blob/main/examples/benchperf/README.md)