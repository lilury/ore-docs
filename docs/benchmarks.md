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
BenchmarkRegisterFunc-20         	 7047313	       145.6 ns/op
BenchmarkRegisterCreator
BenchmarkRegisterCreator-20      	 8499100	       143.6 ns/op
BenchmarkRegisterSingleton
BenchmarkRegisterSingleton-20    	 7971175	       150.8 ns/op
BenchmarkInitialGet
BenchmarkInitialGet-20           	 3168232	       378.3 ns/op
BenchmarkGet
BenchmarkGet-20                  	17260946	        64.98 ns/op
BenchmarkInitialGetList
BenchmarkInitialGetList-20       	 1431822	       839.7 ns/op
BenchmarkGetList
BenchmarkGetList-20              	 5145315	       233.7 ns/op
PASS
```

Checkout also [examples/benchperf/README.md](https://github.com/firasdarwish/ore/blob/main/examples/benchperf/README.md)