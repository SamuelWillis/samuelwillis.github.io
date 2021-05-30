---
title: "Heap Sort, Part 1"
date: 2021-06-01T09:00:00-06:00
draft: false
toc: false
images:
tags:
  - Elixir
  - Algorithms
  - DataStructures
  - Implementation
  - Code
  - Sorting
  - Heapsort
  - Heaps
  - Functional
---
This is the first of a multi-part series on Heapsort.

While the Heapsort algorithm itself is a fairly simple algorithm, it poses some
complexity when implemented in an immutable language that does not support index
based arrays, such as Elixir.

This is because of the Heap data structure that Heapsort relies on.
In imperative languages a Heap is trivially implemented through the use of an
array and calculated indexes. Elixir does not provide us with this luxury,
however.

In this series of posts I will showcase an implementation of Heaps using
pointers that maintains the `O(log(n))` insertion and removal of elements.
It will then be used to implement Heapsort.

The series will be broken up into 4 parts:
1. Introduction to Heap Data Structure (this post)
3. `O(log(n))` insertions with Heap
4. `O(log(n))` removals with Heap
5. Heapsort implementation using the Heap


## Citations
Before we proceed, I would like to give cite some resources that I used to write
this code.

Without these resources, I would probably still be scratching my head in
confusion and I found them to be indispensable while working this Heapsort
implementation.
Especially for the insertion and removal procedures in `O(log(n))` time.

Each resource is also cited in the final implementation's source code.

1. [Paul Picazo's paper on
   Peaps](https://www.cpp.edu/~ftang/courses/CS241/notes/Building_Heaps_With_Pointers.pdf)
    - Provided a much simplified algorithm to find the next place to insert a new
      element
2. Vladimir Kostyukov's [Scala Heap Blog Post](https://kostyukov.net/posts/designing-a-pfds/)
    - A great write up on transforming imperative/RAM based algorithms into
      functional ones.
    - Focussed on implementing a Heap in Scala
3. Vladimir Kostyukov's [paper](http://arxiv.org/pdf/1312.4666v1.pdf)
    - A more complete version of their blog post
    - Includes efficient algorithm for element removal
4. Vladimir Kostyukov's [source code](https://github.com/vkostyukov/scalacaster/blob/master/src/heap/StandardHeap.scala)
    - Scala source code for their own implementation.

Thank you Paul and Vladimir for such great source material.

## Heap Sort
Heap sort uses the properties of Heaps to sort an array.

It works as follows:
1. Place all elements of array into a heap
1. Repeatedly remove the root element of the heap and insert it into the array

The _Heap Property_ ensures that the resulting array is in sorted order.

## Implementation
To start the implementation of heap sort, we will need an efficient heap.
This will be represented by a module with schema in Elixir.

### The heap data structure
With the house-keeping out of the way let's introduce the heap data structure.

There are two properties that a Heap has:
1. **Shape Property:** Heaps are _complete binary trees_. Meaning all levels, except possibly the
   last one, are fully filled and all nodes are filled from left to right.
1. **Heap Property:** The key stored in each node is either greater than or equal to (_max-heap_) or less than or
   equal to (_min-heap_) the keys in the node's children

Some other useful properties are:
1. The left child index of a node in an array representation of a heap is given
   by `2*i` where `i` is the index of the node in the array.
1. The right child index of a node in an array representation of a heap is given
   by `2*i + 1` where `i` is the index of the node in the array.
1. The maximum number of nodes in a complete binary tree is `2^(h) - 1` where
   `h` is the height of the tree. A tree of only a root node has a height of `0`.

### Code
This implementation of a heap will be recursive.
Meaning each node in the heap will itself be a heap.

In the heap we will want to track several things:
1. The **value** of the heap
1. The **size** of the heap, ie: the number of nodes in the heap
1. The **height** of the heap, ie: the largest number of edges from a leaf to
   the node.
1. The **left child** of the heap
1. The **right child** of the heap

To start this implementation, lets configure a struct that will represent a
`Heap`.

The struct will define the supported fields, with some sensible defaults for
empty heaps, as well as a _typespec_.

In an empty Heap, the value and children will be `nil` and the `height` and
`size` will be `0`.

The typespec for the Heap will type the `value` as an integer for simplicities'
sake.

We will also add a simple function called `new` that will return a new `Heap`
for us.
While not necessary, it's a nice to have.

With this in mind we get:
```elixir
defmodule ElixirImpl.DataStructure.Heap do
  @moduledoc """
  A functional implementation of the heap data structure.
  """

  defstruct
  value: nil,
  left: nil,
  size: 0,
  height: 0

  @type t :: %__MODULE__{
          value: integer() | nil
          left: t() | nil,
          right: t() | nil,
          size: integer(),
          height: integer(),
  }

  @doc """
  Builds a new Heap
  """
  @spec new() :: t()
  def new, do: %__MODULE__{}
end
```

## Conclusion
There was a lot of theory covered in this post.
Covering this theory will be extremely useful when the `insert` and `removal`
procedures are implemented.
For each of those operations we will leverage the Heap's properties to make sure
we implement each operation in `O(log(n))` time while also maintaining the heap
properties at each step.

Once that is complete, heapsort will be trivial to implement and we will have
worst case running time of `O(log(n))` ensured.
implement heapsort.

In the next post, we will implement the `O(log(n))` insertion of elements. Stay
tuned until then!
