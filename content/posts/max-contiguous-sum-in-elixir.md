---
title: "Max Subarray in Elixir"
date: 2021-05-20T21:00:50-06:00
draft: false
toc: false
images:
tags:
  - Elixir
  - Algorithms
  - DivideAndConquer
  - Implementation
  - Code
  - MaxiumSubarray
  - ContiguousSum
---
The maximum subarray problem is defined as follows:

Given an array _A_ find a nonempty, contiguous subarray of _A_ whose values have the largest sum.

It's an interesting little problem that benefits greatly from the use of a divide and conquer algorithm as the runtime goes from `O(n^2)` for the brute force method to `O(nlog(n))` for the divide and conquer method.

## Brute Force Method
The brute force method is quite simple.

Take every pair of indices `x` and ` y` s.t. `x < y` and find the sum of elements given by the subarray `A[x..y]`.

Since there is `nC2` ways to chose the pairs of indices, the brute force method is `O(n^2)`.

This can be drastically improved by divide and conquer.

## Divide and Conquer Method
In the method, the array is split into `A[lo..mid]` and `A[mid+1..hi]`.
With this split any contiguous subarray must land in one of three places:
* Entirely in the subarray `A[lo..mid]`
* Entirely in the subarray `A[mid+1..hi]`
* Crossing the midpoint s.t. `low <= i <= mid < j <= hi`

Thus, a maximum subarray must live in exactly one of those places as well.

Finding the maximum subarray of `A[lo..mid]` and `A[mid+1..hi]` can each be done recursively.

The maximum subarray crossing the midpoint will be composed of two arrays `A[i..mid]` and `A[mid+1..j]` where `lo <= i <= mid` and `mid < j <= hi`.

We then compare the sums of each of the 3 maximum subarrays and return the largest sum.

### Runtime of Divide and Conquer Method
Here is a quick explanation for the runtime of the Divide and Conquer method.
It is not in-depth at all and I would recommend looking elsewhere for more complete analysis.
This is based off the [CLRS Algorithms book](https://en.wikipedia.org/wiki/Introduction_to_Algorithms) analysis and is a very condensed summary.


Since the array is split in half for each recursive call and the base cases are `O(1)`, the recursion equation is
```
T(n) = O(1) if n=1
     = 2T(n/2) + O(n) if n > 1
```
Meaning that the [Master Theorem](https://www.programiz.com/dsa/master-theorem) can be applied to solve the equation and the runtime is `O(nlog(n))`.

## Implementation
This implementation will return the sum of maximum subarray.
Since it is recursive, it lends itself nicely to implementation in Elixir.
Returning the subarray can also be done with some adjustments to the code, for simplicity it was omitted.

First we will start with base cases of the recursion:
* For an empty list, the maximum contiguous sum will be `0`.
* For a single element list, the max will be the value of the single element.

```elixir
defmodule ElixirImpl.DivideAndConquer.MaxContiguousSum do
  @moduledoc """
  Find the maximum contiguous sum in a provided list
  """

  alias ElixirImpl.ListHelpers

  @doc """
  Find the maximum contiguous sum in the provided list.
  """
  @spec find(list()) :: integer()
  def find([]), do: 0
  def find([el]), do: el

  def find(list) do
    # TODO
  end
end
```

Next we will need a way to split the array into two parts.
In practice, I'd recommend using [Enum.split/2](https://hexdocs.pm/elixir/Enum.html#split/2) but I found implementing my own to be a nice exercise.

To perform the split, the middle index is calculated, then the original list is split into two recursively with elements `0..middle` in the first array and the remaining elements in the second.

To avoid appending to a list, the left list is built by prepending and then reversed.
Again, a home brewed reverse is used as a learning tool but I'd suggest using [Enum.reverse/1](https://hexdocs.pm/elixir/Enum.html#reverse/1) in practice.
```elixir
  @doc """
  Split a list into two parts leaving count elements in the first part.

  ## Examples

    iex> split([1, 2, 3, 4], 2)
    {[1, 2] ,[3, 4]}

    iex> split([1, 2, 3], 10)
    {[1, 2, 3], []}

    iex> split([1, 2, 3], 0)
    {[], [1, 2, 3]}

    iex> split([], 1)
    {[], []}
  """
  @spec split(list(), integer) :: {list(), list()}
  def split(list, count), do: do_split({[], list}, count)

  def do_split({left, [head | tail]}, count) when 0 < count,
    do: do_split({[head | left], tail}, count - 1)

  def do_split({left, right}, 0), do: {reverse(left), right}

  def do_split({left, []}, _count), do: {reverse(left), []}

  @doc """
  Reverse a list

  ## Examples
    iex> reverse([])
    []

    iex> reverse([1, 2, 3])
    [3, 2, 1]
  """
  @spec reverse(list()) :: list()
  def reverse(list), do: do_reverse([], list)

  def do_reverse([], [head | tail]), do: do_reverse([head], tail)
  def do_reverse(list, [head | tail]), do: do_reverse([head | list], tail)
  def do_reverse(list, []), do: list
```

Adding the middle index calculation, the split, and the recursive calls to `find` it now looks like:
```elixir
  def find(list) do
    middle = list |> length() |> div(2)

    {left, right} = split(list, middle)

    left_sum = find(left)
    right_sum = find(right)
  end
```

The final piece is finding the maximum sum that crosses the middle.
In order to do this, the maximum sum in the array `A[lo..mid]` is found and then added to the maximum sum from the array going from `A[mid+1..hi]`.

To find these maximum sums another recursive function is needed.
This function will take a list and add elements together until the sum is no longer smaller than the sum plus the next element.

Using pattern matching and guards this is quite simple.
Care must be taken when finding the sum of the `left` array as the maximum sum must include the middle element, so it is reversed before being passed to the `find_sum/2` helper.

```elixir
  defp find_cross_sum(left, right) do
    left_sum = left |> ListHelpers.reverse() |> find_sum()
    right_sum = find_sum(right)

    left_sum + right_sum
  end

  defp find_sum(list, sum \\ nil)
  defp find_sum([head | tail], nil), do: find_sum(tail, head)
  defp find_sum([head | tail], sum) when sum <= sum + head, do: find_sum(tail, sum + head)
  defp find_sum(_list, sum), do: sum
```

The find function needs to be updated to find the cross sum, making it look as so:
```elixir
  def find(list) do
    middle = list |> length() |> div(2)

    {left, right} = ListHelpers.split(list, middle)

    left_sum = find(left)
    right_sum = find(right)
    cross_sum = find_cross_sum(left, right)
  end
```

The final piece is returning the maximum of `left_sum`, `right_sum`, and `cross_sum`.
This can be done with a `cond` statement but I thought a `max/3` that leverages guards was nicer to read and easier to follow.

```elixir
  defp max(left_sum, right_sum, cross_sum) when left_sum <= right_sum and cross_sum <= right_sum,
    do: right_sum

  defp max(left_sum, right_sum, cross_sum) when right_sum <= left_sum and cross_sum <= left_sum,
    do: left_sum

  defp max(_left_sum, _right_sum, cross_sum), do: cross_sum
```

Putting this all together the final implementation is

```elixir
defmodule ElixirImpl.DivideAndConquer.MaxContiguousSum do
  @moduledoc """
  Find the maximum contiguous sum in a provided list
  """

  alias ElixirImpl.ListHelpers

  @doc """
  Find the maximum contiguous sum in the provided list.
  """
  @spec find(list()) :: integer()
  def find([]), do: 0
  def find([el]), do: el

  def find(list) do
    middle = list |> length() |> div(2)

    {left, right} = split(list, middle)

    left_sum = find(left)
    right_sum = find(right)
    cross_sum = find_cross_sum(left, right)

    max(left_sum, right_sum, cross_sum)
  end

  defp find_cross_sum(left, right) do
    left_sum = left |> reverse() |> find_sum()
    right_sum = find_sum(right)

    left_sum + right_sum
  end

  # Find max sum helper
  @spec find_sum(list(), nil | integer()) :: integer()
  defp find_sum(list, sum \\ nil)
  defp find_sum([head | tail], nil), do: find_sum(tail, head)
  defp find_sum([head | tail], sum) when sum <= sum + head, do: find_sum(tail, sum + head)
  defp find_sum(_list, sum), do: sum

  # Max of 3 integer helper
  @spec max(integer(), integer(), integer()) :: integer()
  defp max(left_sum, right_sum, cross_sum) when left_sum <= right_sum and cross_sum <= right_sum,
    do: right_sum

  defp max(left_sum, right_sum, cross_sum) when right_sum <= left_sum and cross_sum <= left_sum,
    do: left_sum

  defp max(_left_sum, _right_sum, cross_sum), do: cross_sum

  # Split list helper
  @spec split(list(), integer) :: {list(), list()}
  defp split(list, count), do: do_split({[], list}, count)

  defp do_split({left, [head | tail]}, count) when 0 < count,
    do: do_split({[head | left], tail}, count - 1)

  defp do_split({left, right}, 0), do: {reverse(left), right}

  defp do_split({left, []}, _count), do: {reverse(left), []}

  # Reverse list helper
  @spec reverse(list()) :: list()
  def reverse(list), do: do_reverse([], list)

  defp do_reverse([], [head | tail]), do: do_reverse([head], tail)
  defp do_reverse(list, [head | tail]), do: do_reverse([head | list], tail)
  defp do_reverse(list, []), do: list
end
```

## Conclusion
Finding the maximum contiguous sum in Elixir is quite simple to do.
Especially after wrapping ones head around the recursion involved.

I think that this is easier to follow than the imperative version as well.

I hope this was useful for anyone looking into algorithms in functional languages!

[Click for the full
implementation](https://github.com/SamuelWillis/algorithms/blob/main/elixir/lib/divide_and_conquer/max_contiguous_sum.ex).

[And here is my full set of Insertion Sort
notes](https://github.com/SamuelWillis/algorithms/tree/main/notes/divide-and-conquer/maximum-subarray).
