# deque

This is a fork of [github.com/gammazero/deque](https://github.com/gammazero/deque) to switch to Generics in order to improve performance (Go 1.18+). Here are the results:

### no generics

```
goos: darwin
goarch: amd64
cpu: VirtualApple @ 2.50GHz
BenchmarkPushFront-10        	   10000	    25.81 ns/op
BenchmarkPushBack-10         	   10000	    40.87 ns/op
BenchmarkSerial-10           	   10000	    40.05 ns/op
BenchmarkSerialReverse-10    	   10000	    30.43 ns/op
BenchmarkRotate-10           	   10000	    30122 ns/op
BenchmarkInsert-10           	   10000	    29192 ns/op
BenchmarkRemove-10           	   10000	    13927 ns/op
BenchmarkYoyo-10             	   10000	    1801284 ns/op
BenchmarkYoyoFixed-10        	   10000	    1136005 ns/op
```

### generics (2-3x faster)

```
goos: darwin
goarch: amd64
cpu: VirtualApple @ 2.50GHz
BenchmarkPushFront-10        	   10000	    9.846 ns/op
BenchmarkPushBack-10         	   10000	    10.04 ns/op
BenchmarkSerial-10           	   10000	    11.08 ns/op
BenchmarkSerialReverse-10    	   10000	    26.22 ns/op
BenchmarkRotate-10           	   10000	    11047 ns/op
BenchmarkInsert-10           	   10000	    15644 ns/op
BenchmarkRemove-10           	   10000	    5203 ns/op
BenchmarkYoyo-10             	   10000	    561729 ns/op
BenchmarkYoyoFixed-10        	   10000	    393723 ns/op
```

[![GoDoc](https://pkg.go.dev/badge/github.com/sekoyo/deque)](https://pkg.go.dev/github.com/sekoyo/deque)
[![Build Status](https://github.com/sekoyo/deque/actions/workflows/go.yml/badge.svg)](https://github.com/sekoyo/deque/actions/workflows/go.yml)
[![Go Report Card](https://goreportcard.com/badge/github.com/sekoyo/deque)](https://goreportcard.com/report/github.com/sekoyo/deque)
[![codecov](https://codecov.io/gh/sekoyo/deque/branch/master/graph/badge.svg)](https://codecov.io/gh/sekoyo/deque)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Fast ring-buffer deque ([double-ended queue](https://en.wikipedia.org/wiki/Double-ended_queue)) implementation.

For a pictorial description, see the [Deque diagram](https://github.com/sekoyo/deque/wiki)

## Installation

```
$ go get github.com/sekoyo/deque
```

## Deque data structure

Deque generalizes a queue and a stack, to efficiently add and remove items at either end with O(1) performance. [Queue](<https://en.wikipedia.org/wiki/Queue_(abstract_data_type)>) (FIFO) operations are supported using `PushBack()` and `PopFront()`. [Stack](<https://en.wikipedia.org/wiki/Stack_(abstract_data_type)>) (LIFO) operations are supported using `PushBack()` and `PopBack()`.

## Ring-buffer Performance

This deque implementation is optimized for CPU and GC performance. The circular buffer automatically re-sizes by powers of two, growing when additional capacity is needed and shrinking when only a quarter of the capacity is used, and uses bitwise arithmetic for all calculations. Since growth is by powers of two, adding elements will only cause O(log n) allocations.

The ring-buffer implementation improves memory and time performance with fewer GC pauses, compared to implementations based on slices and linked lists. By wrapping around the buffer, previously used space is reused, making allocation unnecessary until all buffer capacity is used. This is particularly efficient when data going into the dequeue is relatively balanced against data coming out. However, if size changes are very large and only fill and then empty then deque, the ring structure offers little benefit for memory reuse. For that usage pattern a different implementation may be preferable.

For maximum speed, this deque implementation leaves concurrency safety up to the application to provide, however the application chooses, if needed at all.

## Reading Empty Deque

Since it is OK for the deque to contain a nil value, it is necessary to either panic or return a second boolean value to indicate the deque is empty, when reading or removing an element. This deque panics when reading from an empty deque. This is a run-time check to help catch programming errors, which may be missed if a second return value is ignored. Simply check Deque.Len() before reading from the deque.

## Example

```go
package main

import (
    "fmt"
    "github.com/sekoyo/deque"
)

func main() {
    var q deque.Deque
    q.PushBack("foo")
    q.PushBack("bar")
    q.PushBack("baz")

    fmt.Println(q.Len())   // Prints: 3
    fmt.Println(q.Front()) // Prints: foo
    fmt.Println(q.Back())  // Prints: baz

    q.PopFront() // remove "foo"
    q.PopBack()  // remove "baz"

    q.PushFront("hello")
    q.PushBack("world")

    // Consume deque and print elements.
    for q.Len() != 0 {
        fmt.Println(q.PopFront())
    }
}
```

## Uses

Deque can be used as both a:

- [Queue](<https://en.wikipedia.org/wiki/Queue_(abstract_data_type)>) using `PushBack` and `PopFront`
- [Stack](<https://en.wikipedia.org/wiki/Stack_(abstract_data_type)>) using `PushBack` and `PopBack`
