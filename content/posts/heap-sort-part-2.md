---
title: "Heap Sort, Part 2"
date: 2021-07-01T09:00:00-06:00
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
Following from [Part One]({{<ref "posts/heap-sort-part-1">}}) in this heapsort
series, this post will go over `O(log(n))` insertions into the Heap.

Insertion will work using two traversals of the heap, the first finding the
next available leaf and the second bubbling the new value up the heap to
maintain the max heap property.

Due to Elixir's immutability a new heap will need to be built at each step that
updates the heap's height, size, and children.

Since there is two traversals, I'll split the implementation up into two
portions.
The first traversal down to the next available leaf and then the second
traversal to bubble the new value to its correct spot in the heap.

## Finding the leaf
Finding the next available leaf was probably one of the trickier parts of the
insertion.
In the end, it hinges on the array definition of a heap and the index of a
leaf's parent in the array.

To start, we will do the 3 base clauses:
1. Insertion into empty heap => set the heap's value.
2. Insertion into a heap that is a leaf => add a left child to the parent heap
   that has the value being inserted.
3. Insertion into a heap that does not have a right child => add a right child
   to the parent heap that has the value being inserted.

These clauses are the simple cases and cover "filling" a heap.
To implement them a custom guard will be used to determine if a heap is a leaf
or not.
A heap is considered a leaf if it has no children.

In each clause, a new heap will be created that includes the new value at the
appropriate place and the new size of the heap.

Notice only the clauses that add a value to an empty heap or the left child will
increase the heap's height, however.
This is because adding a right child to a heap does not increase the heap's
height.

```elixir
  defguard is_empty(heap) when heap.size == 0

  defguard is_leaf(heap) when is_nil(heap.left) and is_nil(heap.right)

  @doc """
  Push the new value into the heap.
  """
  @spec push(t(), integer()) :: t()
  def push(%__MODULE__{} = heap, value) when is_empty(heap),
    do: %__MODULE__{value: value, height: 1, size: 1}

  def push(%__MODULE__{height: height, size: size} = heap, value)
    when is_leaf(heap),
    do:
      %__MODULE__{
        heap
        | height: height + 1,
          size: size + 1,
          # Push the value onto an empty heap
          left: push(%__MODULE__{}, value)
      }

  def push(%__MODULE__{size: size, left: left, right: right} = heap, value)
    when is_nil(right),
    do:
      %__MODULE__{
        heap
        | size: size + 1,
          # push the value onto a empty heap
          right: push(%__MODULE__{}, value)
      }
```
The remaining two clauses need to determine if the next available spot is in the
left or right child heaps.

The trick to this is finding the next available leaf's parent index if it were
an array implementation.

Given a the size of the heap with the new value inserted, the node will be in
the left subtree if the parent index is divisible by 2, due to the array
index definitions `left_child_index(i): 2*i` and `parent_index(i): floor(i/2)`.

Similarily, the new node will be in the right subtree if the parent index is not
divisible by 2, again from the array index definitions `right(i): 2*i + 1` and
`parent(i): floor(i/2)`.

So the last two clauses will build a new heap with the new value inserted into
the appropriate subtree.
The new heap will have the updated size, as well as a new height.
The height is determined to be the max height of the heap's subtrees plus 1.

Using guards to makes this really expresive:

```elixir
  defguard is_next_leaf_in_left_subtree(size) when size |> div(2) |> rem(2) == 0
  defguard is_next_leaf_in_right_subtree(size) when size |> div(2) |> rem(2) == 1

  @spec push(t(), integer()) :: t()
  def push(%__MODULE__{left: left, right: right, size: size} = heap, value)
      when is_next_leaf_in_left_subtree(size + 1) do
    new_left = push(left, value)

    %__MODULE__{
      heap
      | height: max(new_left.height, right.height) + 1,
        size: size + 1,
        left: new_left,
        right: right
    }
  end

  def push(%__MODULE__{left: left, right: right, size: size} = heap, value)
      when is_next_leaf_in_right_subtree(size + 1) do
    new_right = push(right, value)

    %__MODULE__{
      heap
      | height: max(left.height, new_right.height) + 1,
        size: size + 1,
        left: left,
        right: new_right
    }
  end
```

Now that we've handled pushing a new value onto the heap at the correct place,
we need to ensure that the max-heap property is maintained.

To do this we need to bubble up the inserted value to the correct position in
the heap it was inserted into.

## Bubble up routine
To bubble the new value up to its proper position in the heap a bubble up
routine will be needed.

The routine will accept a Heap struct and check the root value against the
children values, swapping them if needed to maintain the mx heap property.

There are 3 cases:
1. The right value is larger than the left value  and root value
2. The left value is larger than the right value and root value
3. The root is the largest value

In case 1, the root value is swapped with the right value, in case 2 the root
value is swapped with the left value, and in case 3 nothing is swapped and the
original heap is returned.

Implementing this can be written very clearly using function clauses, pattern
matching, and guards in Elixir.

```elixir
  @spec bubble_up(heap :: t()) :: t()
  def bubble_up(__MODULE__{left: __MODULE__{}, right: __MODULE__{}} = heap)
      when heap.value < heap.right.value and
             heap.left.value < heap.right.value,
      do: %__MODULE__{
        heap
        | value: heap.right.value,
          right: __MODULE__{heap.right | value: heap.value}
      }

  def bubble_up(__MODULE__{left: __MODULE__{}} = heap)
      when heap.value < heap.left.value,
      do: %__MODULE__{
        heap
        | value: heap.left.value,
          left: __MODULE__{heap.left | value: heap.value}
      }

  def bubble_up(%__MODULE__{} = heap), do: heap
```

Now the routine needs to be used by the `push` function.

### Putting it all together
Putting all the above together the resulting code is quite lengthy but still
expressive in its intent.

```elixir
defmodule ElixirImpl.DataStructure.Heap do
  defguard is_empty(heap) when heap.size == 0

  defguard is_leaf(heap) when is_nil(heap.left) and is_nil(heap.right)

  defguard is_next_leaf_in_left_subtree(size) when size |> div(2) |> rem(2) == 0
  defguard is_next_leaf_in_right_subtree(size) when size |> div(2) |> rem(2) == 1

  @doc """
  Push the new value into the heap.
  """
  @spec push(t(), integer()) :: t()
  def push(%__MODULE__{} = heap, value) when is_empty(heap),
    do: %__MODULE__{value: value, height: 1, size: 1}

  def push(%__MODULE__{height: height, size: size} = heap, value)
    when is_leaf(heap),
    do:
      bubble_up(%__MODULE__{
        heap
        | height: height + 1,
          size: size + 1,
          left: push(%__MODULE__{}, value)
      })

  def push(%__MODULE__{size: size, left: left, right: right} = heap, value)
    when is_nil(right),
    do:
      bubble_up(%__MODULE__{
        heap
        | size: size + 1,
          right: push(%__MODULE__{}, value)
      })

  def push(%__MODULE__{left: left, right: right, size: size} = heap, value)
      when is_next_leaf_in_left_subtree(size + 1) do
    new_left = push(left, value)

    bubble_up(%__MODULE__{
      heap
      | height: max(new_left.height, right.height) + 1,
        size: size + 1,
        left: new_left,
        right: right
    })
  end

  def push(%__MODULE__{left: left, right: right, size: size} = heap, value)
      when is_next_leaf_in_right_subtree(size + 1) do
    new_right = push(right, value)

    bubble_up(%__MODULE__{
      heap
      | height: max(left.height, new_right.height) + 1,
        size: size + 1,
        left: left,
        right: new_right
    })
  end

  @spec bubble_up(heap :: t()) :: t()
  def bubble_up(__MODULE__{left: __MODULE__{}, right: __MODULE__{}} = heap)
      when heap.value < heap.right.value and
             heap.left.value < heap.right.value,
      do: %__MODULE__{
        heap
        | value: heap.right.value,
          right: __MODULE__{heap.right | value: heap.value}
      }

  def bubble_up(__MODULE__{left: __MODULE__{}} = heap)
      when heap.value < heap.left.value,
      do: %__MODULE__{
        heap
        | value: heap.left.value,
          left: __MODULE__{heap.left | value: heap.value}
      }

  def bubble_up(%__MODULE__{} = heap), do: heap
end
```

## Conclusion
Although complicated, Elixir really helped with writing clear and intention
revealing code through guards and pattern matching.

I can't help but attribute this to the lack of `if` statements needed inside of
functions, but that's a discussion for another day.

A downside to this code is, although it's intention revealing, it's likely less
perfomant than an iterative implementation.
There's a lot of memory use behind the creation of new heaps each time we need
to update a value.

Elixir's immutability is a hinderance here but it is maybe offset somewhat by
garbage collection being able to dispose of the intermediary heaps as soon as
they're no longer in use.
I do not fully understand garbage collection in Elixir yet, but I believe this
is the case.

If you're interested in the full implementation of a Heap, [see it
here](https://github.com/SamuelWillis/algorithms/blob/main/elixir/lib/data_structures/heap.ex).

If you have comments, improvements, or corrections please feel free to leave an
[issue in Github](https://github.com/SamuelWillis/algorithms/issues).
