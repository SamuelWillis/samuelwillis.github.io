---
title: "Queues in Elixir"
date: 2021-04-13T18:36:07-06:00
publishdate: 2021-04-19T15:10:00-06:00
draft: false
toc: false
images:
tags:
  - Elixir
  - DataStructures
  - Implementation
  - Code
---
Continuing on from my previous [Stacks in Elixir]({{<ref "posts/stacks-in-elixir">}}) post
today I will cover implementing Queues in Elixir.

Queues operate on a First In First out (FIFO) policy, meaning the first item
placed onto the queue is the first to be removed.

These are again essentially sugar for lists but there's a few gotchas to get
around when implementing them in Elixir.
The main being that appending to lists can be an expensive operation in Elixir.

But first, the queue operations.

## Operations
Formally, all the operations for Queues are:

```
Enqueue(Q, x)
  if Queue-Full(Q) throw Overflow Error

  Q[Q.tail] = x
  if Q.tail == Q.length -1
    Q = 0
  else Q.tail++
  return x

Dequeue(Q)
  if Queue-Empty(Q) throw Underflow Error

  x = Q[Q.head]
  if Q.head == Q.length - 1
    Q.head = 0
  else Q.head++
  return x

Queue-Empty(Q)
  return Q.tail == Q.head

Queue-Full(Q)
  if Q.head = Q.tail + 1 OR (Q.tail = Q.length - 1 AND Q.head = 0)
    return true
  else
    return false
```
This implementation will omit the full operation as a list will be used to store
the items on the queue and they do not require manual resizing, unlike arrays
would.

## Implementation
The implementation will again use a list to store the items in the queue.

Normally when adding an item to the queue it would be added to the back.
This would require appending to a list and in Elixir, that can be very
expensive.
Appending the new item would be possible but is [warned
against](https://hexdocs.pm/elixir/Kernel.html#++/2).
The official suggestion is to prepend and then reverse the list.

This implementation will follow the documentation's advice and prepend any new
items to the queue.
Then to dequeue it will reverse the elements and return the last one.

I admit this is probably overkill and appending would probably be fine but I
wanted to write my own reverse function to practice my Elixir.

In practice, appending would likely be fine unless the Queue was expected to be
extremely large and then using the
[`Enum.reverse`](https://hexdocs.pm/elixir/Enum.html#reverse/1) built in functions
would be better than rolling my own as it leverages the highly optimized
`:list.reverse`
from Erlang.

### Set up
As in the [Stack implementation]({{< ref
"posts/stacks-in-elixir#implementation" >}}) a struct will be used to represent
the Queue.

```elixir
defmodule ElixirImpl.DataStructure.Queue do
  @moduledoc """
  Queue implementation.
  """
  defstruct elements: []

  @typep t :: %__MODULE__{
           elements: [any()]
         }
end
```

### Enqueue
Because this implementation places the new items at the front of the list of
elements the enqueue implementation is quite simple.

```elixir
  @doc """
  Add an item into a queue.

  ## Examples
   iex> Queue.enqueue(%Queue{elements: []}, "one")
    %Queue{elements: ["one"]}

    iex> Queue.enqueue(%Queue{elements: ["one"]}, "two")
    %Queue{elements: ["two", "one"]}
  """
  def enqueue(%__MODULE__{} = queue, x) do
    %__MODULE__{
      queue
      | elements: [x | queue.elements]
    }
  end
```

#### Dequeue
Dequeueing is a little trickier as the list of elements will have the oldest
item at the end of the list.

As mentioned in the [Implementation Section]({{< ref
"posts/queues-in-elixir#implementation" >}}) some list reversals will need to be
performed.

There are two reversals total, the first is to get the oldest item, the second
is to put the list back into the "correct" order.

In the case we dequeue an item, a tuple is returned consisting of the `:ok`
atom, the item, and the queue with the list of elements omitting the dequeued
item.

If the list of elements is empty a `{:error, "Empty Queue"}` tuple is returned.

Putting it together it looks as follows:
```elixir
  @doc """
  Remove an item from the queue.

  ## Examples
    iex> Queue.dequeue(%Queue{elements: ["first"]})
    {:ok, "first", %Queue{elements: []}

    iex> Queue.dequeue(%Queue{elements: ["third", "second", "first"]})
    {:ok, "first", %Queue{elements: ["third", "second"]}}

    iex> Queue.dequeue(%Queue{elements: []})
    {:error, "Empty Queue"}
  """
  def dequeue(%__MODULE__{elements: []}), do: {:error, "Empty Queue"}

  # Reverses the list of elements and grabs the "first" queued element and the
  # remainder of the elements. Then reverses the remainder to place them back
  # into their correct order
  def dequeue(%__MODULE__{elements: elements}) do
    [item | elements] = reverse_elements(elements)

    {:ok, item,
     %__MODULE__{
       elements: reverse_elements(elements)
     }}
  end
```

### Empty
The empty operation is quite simple using pattern matching.

If the Stack's elements matches a non empty list, we return `false`.
Otherwise, return `true`.
The [trailing question
mark](https://hexdocs.pm/elixir/master/naming-conventions.html#trailing-question-mark-foo)
naming convention is used in the implementation.

```elixir
  @doc """
  Check the if queue is empty

  ## Examples
    iex> Queue.empty?(%Queue{elements: ["item one"]})
    false

    iex> Queue.empty?(%Queue{elements: []})
    true
  """
  def empty?(%__MODULE__{elements: [_head | _tail]}), do: false
  def empty?(%__MODULE__{elements: []}), do: true
```

## Conclusion
Implementing a simple Queue is a fairly easy task in Elixir.
It could be made even simpler if
[Kernel.++/2](https://hexdocs.pm/elixir/Kernel.html#++/2) was used to append the
new items to the back of the list of elements.

As it stands, implementing a simple Queue was a great way to get familiar with
Elixir's lists and their limitations as well as start digging into idiomatic
ways to handle lists.

[Click for the full
implementation](https://github.com/SamuelWillis/algorithms/blob/main/elixir/lib/data_structures/queue.ex).

[And here is my full set of Queue
notes](https://github.com/SamuelWillis/algorithms/tree/main/notes/data-structures/queue).
