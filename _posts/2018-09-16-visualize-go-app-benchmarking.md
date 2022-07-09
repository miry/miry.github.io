---
url: https://medium.com/notes-and-tips-in-full-stack-development/visualize-go-app-benchmarking-2a3be0169e26
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/visualize-go-app-benchmarking-2a3be0169e26
title: Visualize Go app benchmarking
subtitle: It is interesting to analyze how a data structure has a different performance
  based on number of items or the way of implementation…
slug: visualize-go-app-benchmarking
description: ""
tags:
- benchmark
- google-charts
- go
author: Michael Nikitochkin
username: miry
---

# Visualize Go app benchmarking

![Photo by Volkan Olmez on Unsplash](/assets/2018-09-16-visualize-go-app-benchmarking-1_sKpJmj16sJwoAIkAlhkYgQ.jpeg)

It is interesting to analyze how a data structure has a different performance based on number of items or the way of implementation. Benchmark metrics simplify the review of performance insights. We can use Spreadsheets or Excel to visualize those metrics and guess the complexity level. I figured out how to automate the process using simple but effective approach.

I was looking for a universal solution to show charts and make empirical analysis for some algorithms. Ideal tool should run an application with dynamic arguments and then build the actual chart. Here is the example of a basic version:

```
$ bench -step i*10 -max 100000 -- app <N>
```

In the end, I should see a [Google Chart](https://developers.google.com/chart/) with `Time/N` data. I could not able to find any tool to generate such charts, though. However, I got some custom integrations with benchmark libs in `Ruby` and `Golang`, which can build charts based on `Benchmarks` results.

I want to draw your attention to one small tool available here: [https://github.com/codingberg/benchgraph](https://github.com/codingberg/benchgraph). The client part parse standard Go benchmark results and sends the data to a server. It uses the convention in benchmark names to build function result performance. Here is the example of benchmark code and output:

```
package benchmarks

import (
	"fmt"
	"math/rand"
	"testing"
)

/*
BenchmarkAccessStructure show compare metrics between data strucuture and number of items.
*/
func BenchmarkAccessStructure(b *testing.B) {
	for _, size := range []int{1, 10, 100, 1000, 10000, 100000, 1000000} {
		benchmarkAccessStructure(b, size)
	}
}

func benchmarkAccessStructure(b *testing.B, size int) {
	var indexes = make([]int, size, size)
	var arr = make([]int, size, size)
	var hash = make(map[int]int)

	rand.Seed(size % 42)
	for i := 0; i < size; i++ {
		indexes[i] = rand.Intn(size)
		arr[i] = i
		hash[i] = i
	}

	b.ResetTimer()

	b.Run(fmt.Sprintf("Array_%d", size), func(b *testing.B) {
		for i := 0; i < b.N; i++ {
			indx := indexes[i%size] % size
			_ = arr[indx]
		}
	})

	b.Run(fmt.Sprintf("Hash_%d", size), func(b *testing.B) {
		for i := 0; i < b.N; i++ {
			indx := indexes[i%size] % size
			_ = hash[indx]
		}
	})
}

```
> *[access_test.go view raw](https://gist.githubusercontent.com/miry/4e207a6ca3e0065fc1f1a08ec3938b24/raw/e7d88222918acfaa0761d0ebb97d631661da343c/access_test.go)*

```
$ go test -bench=.
goos: darwin
goarch: amd64
pkg: github.com/miry/samples/benchmarks
BenchmarkAccessStructure/Array_1-4         	100000000	        25.2 ns/op
BenchmarkAccessStructure/Hash_1-4          	50000000	        35.5 ns/op
BenchmarkAccessStructure/Array_10-4        	50000000	        25.5 ns/op
BenchmarkAccessStructure/Hash_10-4         	30000000	        42.8 ns/op
BenchmarkAccessStructure/Array_100-4       	100000000	        25.2 ns/op
BenchmarkAccessStructure/Hash_100-4        	30000000	        43.6 ns/op
BenchmarkAccessStructure/Array_1000-4      	50000000	        25.5 ns/op
BenchmarkAccessStructure/Hash_1000-4       	20000000	        63.6 ns/op
BenchmarkAccessStructure/Array_10000-4     	100000000	        25.3 ns/op
BenchmarkAccessStructure/Hash_10000-4      	20000000	        73.1 ns/op
BenchmarkAccessStructure/Array_100000-4    	50000000	        25.4 ns/op
BenchmarkAccessStructure/Hash_100000-4     	20000000	        95.6 ns/op
BenchmarkAccessStructure/Array_1000000-4   	50000000	        25.3 ns/op
BenchmarkAccessStructure/Hash_1000000-4    	10000000	       169 ns/op
PASS
ok  	github.com/miry/samples/benchmarks	24.523s
```
> *[benchmark.out view raw](https://gist.githubusercontent.com/miry/4e207a6ca3e0065fc1f1a08ec3938b24/raw/fed5a7633867b66c2dc65264a2174492a54e04a3/benchmark.out)*

To generate a chart, we need to install the client and run this script:

```
$ go test -bench=. | benchgraph
```

It would return a link to the chart, here is the example: [http://benchgraph.codingberg.com/48](http://benchgraph.codingberg.com/48).

![](/assets/2018-09-16-visualize-go-app-benchmarking-1_HMU7s4mxlHUJcdgckfEznA.png)

This chart could be embedded to your page with a code snippet that available on the page. It uses Google Charts in the background. You can read sources of the page and build your own online solution.

![](/assets/2018-09-16-visualize-go-app-benchmarking-1_191jZ4P6L-uGpX-7Fs1zGg.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).


