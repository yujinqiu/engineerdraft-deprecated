---
title: "Defer overhead in go"
description: "Defer overhead in go"
date: "2014-05-14"
categories:
    - "go"
---


#### Prelude
This post based on
[real events](https://github.com/dotcloud/docker/commit/5128feb690e8fd0244d1fecef5f3f5f77598bbfa)
in docker repository.
When I revealed that my 20-percent-cooler refactoring made `Pop` function x4-x5
times slower, I did some research and concluded, that problem was in using
`defer` statement for unlocking everywhere.

In this post I'll write simple program and benchmarks from which we will see,
that sometimes `defer` statement can slowdown your program a lot.

Let's create simple queue with methods `Put` and `Get`. Next snippets shows such
queue and benchmarks for it. Also I wrote duplicate methods with `defer` and
without it.

#### Code

{{% highlight go %}}
package defertest

import (
	"sync"
)

type Queue struct {
	sync.Mutex
	arr []int
}

func New() *Queue {
	return &Queue{}
}

func (q *Queue) Put(elem int) {
	q.Lock()
	q.arr = append(q.arr, elem)
	q.Unlock()
}

func (q *Queue) PutDefer(elem int) {
	q.Lock()
	defer q.Unlock()
	q.arr = append(q.arr, elem)
}

func (q *Queue) Get() int {
	q.Lock()
	if len(q.arr) == 0 {
		q.Unlock()
		return 0
	}
	res := q.arr[0]
	q.arr = q.arr[1:]
	q.Unlock()
	return res
}

func (q *Queue) GetDefer() int {
	q.Lock()
	defer q.Unlock()
	if len(q.arr) == 0 {
		return 0
	}
	res := q.arr[0]
	q.arr = q.arr[1:]
	return res
}
{{% /highlight %}}

#### Benchmarks

{{% highlight go %}}
package defertest

import (
	"testing"
)

func BenchmarkPut(b *testing.B) {
	q := New()
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		for j := 0; j < 1000; j++ {
			q.Put(j)
		}
	}
}

func BenchmarkPutDefer(b *testing.B) {
	q := New()
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		for j := 0; j < 1000; j++ {
			q.PutDefer(j)
		}
	}
}

func BenchmarkGet(b *testing.B) {
	q := New()
	for i := 0; i < 1000; i++ {
		q.Put(i)
	}
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		for j := 0; j < 2000; j++ {
			q.Get()
		}
	}
}

func BenchmarkGetDefer(b *testing.B) {
	q := New()
	for i := 0; i < 1000; i++ {
		q.Put(i)
	}
	b.ResetTimer()
	for i := 0; i < b.N; i++ {
		for j := 0; j < 2000; j++ {
			q.GetDefer()
		}
	}
}
{{% /highlight %}}

#### Results

```
BenchmarkPut       50000             63002 ns/op
BenchmarkPutDefer  10000            143391 ns/op
BenchmarkGet       50000             72045 ns/op
BenchmarkGetDefer  10000            249029 ns/op
```

#### Conclusion

You don't need defers in small functions with one-two exit points.

#### Update

Retested with go from `tip` as Cezar Sá Espinola suggested. So, here results:

```
BenchmarkPut       50000             54633 ns/op
BenchmarkPutDefer          10000            102971 ns/op
BenchmarkGet       50000             65148 ns/op
BenchmarkGetDefer          10000            180839 ns/op
```
