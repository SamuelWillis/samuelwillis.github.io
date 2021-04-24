---
title: "Insertion Sort in Elixir"
date: 2021-04-24T12:59:20-06:00
publishdate: 2021-04-25T10:00:00-06:00
draft: false
toc: false
images:
tags:
  - Elixir
  - Algorithms
  - Implementation
  - Code
  - Sorting
  - InsertionSort
---
Departing from Data Structures such as [Queues]({{<ref "posts/queues-in-elixir">}})
and [Stacks]({{<ref "posts/stacks-in-elixir">}}) this post will cover the insertion
sort algorithm in Elixir.

Insertion sort is one of the first sorting algorithms introduced in Computer
Science as its quite simple.

Insertion sort is quite efficient on small data sets and it does the sort in
place.
Meaning it has `O(1)` (constant) memory space requirements.
Unfortunately, this does not hold true in Elixir as recursion is used causing
the space-complexity to be `O(n)` (linear).


## Method
In insertion sort, the array into is split into two "piles".

On the left pile are the sorted elements and on the right pile are the unsorted.

For each element on the right (unsorted), we compare it to elements in the left
(sorted) pile until the element from the right pile is in its sorted position in
the array.

If the left hand element is larger than the right hand element, we move the left
hand element to the "right" one position and repeat.

If the left hand element is less than or equal to the right hand element, we
place the right hand element into the position to the "right" of the current
left hand position.

## Code
In imperative languages, insertion sort is most commonly implemented using for
loops and while loops.

Elixir does not have loops, which can be a little tricky to navigate when you
first start learning Elixir.

Instead of loops, Elixir provides pattern matching and recursion.

Placing the insertion sort implementation into a module, the setup will be as
follows:
```elixir
defmodule ElixirImpl.Sorting.InsertionSort do
  @moduledoc """
  Simple insertion sort implementation
  """

  @doc """
  Sort the provided list

  ## Examples

    iex> InsertionSort.sort([])
    []

    iex> InsertionSort.sort([1])
    [1]

    iex> InsertionSort.sort([1, 0])
    [0, 1]

    iex> InsertionSort.sort([1, -1, 0])
    [-1, 0, 1]
  """
  @spec sort(list()) :: list()
  def sort([]), do: []
  def sort([first]), do: [first]
  def sort([first | rest]), do: do_sort([first], rest)
end
```
Note: only Lists were supported in this implementation for simplicity.

The main sorting logic will come in the form of two private recursive functions.

The first, `do_sort/2` will handle looping over each element of the array and
placing it into the correct position amongst the sorted elements.

To do this an accumulator will be used to store the elements in sorted position.

`do_sort/2` accepts the sorted elements as its first parameter and the remaining
unsorted elements as its second parameter.

It will consist of 2 clauses.
```elixir
  defp do_sort(sorted, _unsorted = [head | tail]), do: do_sort(insert(head, sorted), tail)
  defp do_sort(sorted, []), do: sorted
```

The second private function is `insert/2`.
It takes care of looping through the sorted list and placing an element in the
correct position.

```elixir
  # if no elements to compare against in sorted list, return element as a list
  defp insert(element, _sorted = []), do: [element]
  # if element is less than the first element of the sorted list, insert at front
  defp insert(element, [min | _rest] = sorted) when element <= min, do: [element | sorted]
  # Otherwise element is larger than min and we check it against the next sorted element
  defp insert(element, [min | rest]), do: [min | insert(element, rest)]
```


Putting it all together the implementation is:
```elixir
defmodule ElixirImpl.Sorting.InsertionSort do
  @moduledoc """
  Simple insertion sort implementation
  """

  @doc """
  Sort the provided list

  ## Examples

    iex> InsertionSort.sort([])
    []

    iex> InsertionSort.sort([1])
    [1]

    iex> InsertionSort.sort([1, 0])
    [0, 1]

    iex> InsertionSort.sort([1, -1, 0])
    [-1, 0, 1]
  """
  @spec sort(list()) :: list()
  def sort([]), do: []
  def sort([first]), do: [first]
  def sort([first | rest]), do: do_sort([first], rest)

  defp do_sort(sorted, _unsorted = [head | tail]),
    do: do_sort(insert(head, sorted), tail)
  defp do_sort(sorted, []),
    do: sorted

  # if no elements to compare against in sorted, return element as a list
  defp insert(element, _sorted = []),
    do: [element]
  # if element is less than the first element of the sorted list, insert at front
  defp insert(element, [min | _rest] = sorted) when element <= min,
    do: [element | sorted]
  # Otherwise try insert element into remainder of sorted list
  defp insert(element, [min | rest]),
    do: [min | insert(element, rest)]
end
```

## Conclusion
Insertion sort is a simple sorting algorithm and implementing it in Elixir was a
great way to get familiar with [recursion](https://en.wikipedia.org/wiki/Recursion_(computer_science))
and [tail-call recursion](https://en.wikipedia.org/wiki/Tail_call).
It was also a great exercise to increase comfort with pattern matches and
looping lists.

This implementation did lose the `O(1)` space complexity of an imperative
implementation, but that loss holds true for most recursive implementations as
space is needed on the stack for all the function calls.

[Click for the full
implementation](https://github.com/SamuelWillis/algorithms/blob/main/elixir/lib/sorting/insertion_sort.ex).

[And here is my full set of Insertion Sort
notes](https://github.com/SamuelWillis/algorithms/tree/main/notes/sorting/insertion-sort).
