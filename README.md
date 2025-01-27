# queue

![GitHub release](https://img.shields.io/github/v/tag/adrianbrad/queue)
[![GitHub go.mod Go version of a Go module](https://img.shields.io/github/go-mod/go-version/adrianbrad/queue)](https://github.com/adrianbrad/queue)
[![GoDoc reference example](https://img.shields.io/badge/godoc-reference-blue.svg)](https://pkg.go.dev/github.com/adrianbrad/queue)

[![CodeFactor](https://www.codefactor.io/repository/github/adrianbrad/queue/badge)](https://www.codefactor.io/repository/github/adrianbrad/queue)
[![Go Report Card](https://goreportcard.com/badge/github.com/adrianbrad/queue)](https://goreportcard.com/report/github.com/adrianbrad/queue)
[![codecov](https://codecov.io/gh/adrianbrad/queue/branch/main/graph/badge.svg)](https://codecov.io/gh/adrianbrad/queue)

[![lint-test](https://github.com/adrianbrad/queue/actions/workflows/lint-test.yaml/badge.svg)](https://github.com/adrianbrad/queue/actions?query=workflow%3Alint-test)
[![grype](https://github.com/adrianbrad/queue/actions/workflows/grype.yaml/badge.svg)](https://github.com/adrianbrad/queue/actions?query=workflow%3Agrype)
[![codeql](https://github.com/adrianbrad/queue/actions/workflows/codeql.yaml/badge.svg)](https://github.com/adrianbrad/queue/actions?query=workflow%3ACodeQL)

---

The `queue` package provides thread-safe generic implementations in Go for the following data structures: `BlockingQueue`, `PriorityQueue` and `CircularQueue`.

A queue is a sequence of entities that is open at both ends where the elements are
added (enqueued) to the tail (back) of the queue and removed (dequeued) from the head (front) of the queue.

The implementations are designed to be easy-to-use and provide a consistent API, satisfying the `Queue` interface provided by this package. .

Benchmarks and Example tests can be found in this package. 

<!-- TOC -->
* [queue](#queue)
  * [Installation](#installation)
  * [Import](#import)
  * [Usage](#usage)
    * [Queue Interface](#queue-interface)
    * [Blocking Queue](#blocking-queue)
    * [Priority Queue](#priority-queue)
    * [Circular Queue](#circular-queue)
  * [Benchmarks](#benchmarks-)
<!-- TOC -->

## Installation
To add this package as a dependency to your project, run:

```shell
go get -u github.com/adrianbrad/queue
```

## Import
To use this package in your project, you can import it as follows:

```go
import "github.com/adrianbrad/queue"
```

## Usage

### Queue Interface

```go
// Queue is a generic queue interface, defining the methods that all queues must implement.
type Queue[T comparable] interface {
	Get() (T, error)
	Offer(T) error
	Reset()
	Peek() (T, error)
	Size() int
	IsEmpty() bool
	Iterator() <-chan T
	Clear() []T
}
```

### Blocking Queue

Blocking queue is a FIFO ordered data structure. Both blocking and non-blocking methods are implemented.
Blocking methods wait for the queue to have available items when dequeuing, and wait for a slot to become available in case the queue is full when enqueuing.
The non-blocking methods return an error if an element cannot be added or removed. 
Implemented using sync.Cond from the standard library.

```go
package main

import (
	"fmt"

	"github.com/adrianbrad/queue"
)

func main() {
	elems := []int{2, 3}

	blockingQueue := queue.NewBlocking(elems, queue.WithCapacity(3))

	containsTwo := blockingQueue.Contains(2)
	fmt.Println(containsTwo) // true

	size := blockingQueue.Size()
	fmt.Println(size) // 2

	empty := blockingQueue.IsEmpty()
	fmt.Println(empty) // false

	if err := blockingQueue.Offer(1); err != nil {
		// handle err
	}

	elem, err := blockingQueue.Get()
	if err != nil {
		// handle err
	}

	fmt.Println("elem: ", elem) // elem: 2
}
```

### Priority Queue

Priority Queue is a data structure where the order of the elements is given by a comparator function provided at construction. 
Implemented using container/heap standard library package.

```go
package main

import (
	"fmt"

	"github.com/adrianbrad/queue"
)

func main() {
	elems := []int{2, 3, 4}

	priorityQueue := queue.NewPriority(
		elems, 
		func(elem, otherElem int) bool { return elem < otherElem },
        )

	containsTwo := priorityQueue.Contains(2)
	fmt.Println(containsTwo) // true

	size := priorityQueue.Size()
	fmt.Println(size) // 3

	empty := priorityQueue.IsEmpty()
	fmt.Println(empty) // false

	if err := priorityQueue.Offer(1); err != nil {
		// handle err
	}

	elem, err := priorityQueue.Get()
	if err != nil {
		// handle err
	}

	fmt.Printf("elem: %d\n", elem) // elem: 1
}
```

### Circular Queue

Circular Queue is a fixed size FIFO ordered data structure. When the queue is full, adding a new element to the queue overwrites the oldest element.

Example:
We have the following queue with a capacity of 3 elements: [1, 2, 3].
If the tail of the queue is set to 0, as if we just added the element `3`,
the next element to be added to the queue will overwrite the element at index 0.
So, if we add the element `4`, the queue will look like this: [4, 2, 3].
If the head of the queue is set to 0, as if we never removed an element yet,
then the next element to be removed from the queue will be the element at index 0, which is `4`.

```go
package main

import (
  "fmt"

  "github.com/adrianbrad/queue"
)

func main() {
  elems := []int{2, 3, 4}

  circularQueue := queue.NewCircular(elems, 3)

  containsTwo := circularQueue.Contains(2)
  fmt.Println(containsTwo) // true

  size := circularQueue.Size()
  fmt.Println(size) // 3

  empty := circularQueue.IsEmpty()
  fmt.Println(empty) // false

  if err := circularQueue.Offer(1); err != nil {
    // handle err
  }

  elem, err := circularQueue.Get()
  if err != nil {
    // handle err
  }

  fmt.Printf("elem: %d\n", elem) // elem: 1
}
```

## Benchmarks 

Results as of April 2023.

```text
BenchmarkBlockingQueue/Peek-12          72900492                18.92 ns/op            0 B/op          0 allocs/op
BenchmarkBlockingQueue/Get_Offer-12     14937858                95.08 ns/op           41 B/op          0 allocs/op
BenchmarkBlockingQueue/Offer-12         26680512                51.81 ns/op           45 B/op          0 allocs/op
BenchmarkCircularQueue/Peek-12          73749498                16.24 ns/op            0 B/op          0 allocs/op
BenchmarkCircularQueue/Get_Offer-12     18768650                63.02 ns/op            0 B/op          0 allocs/op
BenchmarkCircularQueue/Offer-12         38328231                37.57 ns/op            0 B/op          0 allocs/op
BenchmarkPriorityQueue/Peek-12          75156879                15.79 ns/op            0 B/op          0 allocs/op
BenchmarkPriorityQueue/Get_Offer-12     17643837                68.65 ns/op            0 B/op          0 allocs/op
BenchmarkPriorityQueue/Offer-12         20506784                57.43 ns/op           54 B/op          0 allocs/op
```