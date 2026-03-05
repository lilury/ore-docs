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
BenchmarkRegisterFunc-20         	 4785020	       210.3 ns/op
BenchmarkRegisterCreator
BenchmarkRegisterCreator-20      	 6592044	       184.9 ns/op
BenchmarkRegisterSingleton
BenchmarkRegisterSingleton-20    	 6123404	       214.0 ns/op
BenchmarkInitialGet
BenchmarkInitialGet-20           	 2663497	       431.1 ns/op
BenchmarkGet
BenchmarkGet-20                  	11069119	       110.3 ns/op
BenchmarkInitialGetList
BenchmarkInitialGetList-20       	 1317560	       914.6 ns/op
BenchmarkGetList
BenchmarkGetList-20              	 4091818	       284.5 ns/op
PASS
```

Checkout also [examples/benchperf/README.md](https://github.com/firasdarwish/ore/blob/main/examples/benchperf/README.md)