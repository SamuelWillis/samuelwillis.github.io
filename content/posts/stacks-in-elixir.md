---
title: "Stacks in Elixir"
date: 2021-04-11T15:12:45-06:00
draft: false
toc: false
images:
tags:
  - Elixir
  - DataStructures
  - Implementation
  - Code
---
Stacks are one of the most basic data structures in Computer Science.
Nonetheless, they are quite useful and used often.

They operate under the Last In First Out (LIFO) policy, meaning the last object
placed onto the stack is the first one to be removed from it.

In Elixir they are essentially syntactical sugar for
[Lists](https://hexdocs.pm/elixir/List.html) but implementing them was a nice
way to get acquainted with [Structs](https://elixir-lang.org/getting-started/structs.html)
and basic [pattern
matching](https://elixir-lang.org/getting-started/pattern-matching.html).

## Operations
Formally, the operations for a Stack are **push**, **pop**, **empty**, and **full**.
They typically include checks for _overflow errors_ (pushing to a full stack) and
_underflow errors_ (popping from an empty stack).

```
Push(S, x)
  if Full(S) throw Overflow Error
  S[++S.top] = x

Pop(S)
  if Empty(S) throw Underflow Error
  return S[--S.top]

Empty(S)
  if count(S) == 0:
    return True
  else
    return False

Full(S)
  if S.top == S.length
    return True
  else
    return False
```

## Implementation
The Elixir implementation will use a linked list.
This is because we do not have access to a traditional/imperative style arrays in
Elixir.

Linked lists do not have a static size and, as such, they do not require
resizing when too many elements are added to the stack.
Because of this each operation will take constant time in the **worst
case**.
This does come at the cost of more space used to store each item.

A [Struct](https://elixir-lang.org/getting-started/structs.html) will be used to
represent a Stack.
A field called `elements` will be used to store the list of elements.
Each item in the elements list can have any type.


```elixir
defmodule ElixirImpl.DataStructure.Stack do
  @moduledoc """
  Stack implementation.

  This is essentially sugar for Elixir Lists, but is here to show how basic
  operations should work on a stack.
  """
  defstruct elements: []

  @typep t :: %__MODULE__{
           elements: [any()]
         }
end
```

### Push
Lets add the push operation.

Since the elements struct key will always exist the `|` operator to can be
used to update the elements key in the struct without worrying about an error.

Thus the `push` operation looks like so:
```elixir
  @doc """
  Push an item onto a stack.

  ## Examples
    iex> Stack.push(%Stack{elements: []}, "one")
    %Stack{elements: ["one"]}

    iex> Stack.push(%Stack{elements: ["one"]}, "two")
    %Stack{elements: ["two", "one"]}
  """
  @spec push(__MODULE__.t(), any()) :: __MODULE__.t()
  def push(%__MODULE__{} = stack, x) do
    %__MODULE__{
      stack
      | elements: [x | stack.elements]
    }
  end
```

### Pop
Next, the **pop** operation.

For the pop, a check for a empty elements list will be needed.

This check is done using pattern matching.
If the Stack's elements matches an empty list, an error tuple is returned with a
reason.

Pattern matching is also used when the elements list is non empty.
In this case, the `head` and `tail` of the elements list is matched against.
The Stack then has its elements updated to be the matched `tail` and a tuple is
returned consisting of a `:ok` atom, the matched `head`, and the updated Stack.

```elixir
  @doc """
  Pop an item off the top of the stack

  ## Examples
    iex> Stack.pop(%Stack{elements: ["item one"]})
    {:ok, "item one", %Stack{elements: []}}

    iex> Stack.pop(%Stack{elements: ["item two", "item one"]})
    {:ok, "item two", %Stack{elements: ["item one"]}}

    iex> Stack.pop(%Stack{elements: []})
    {:error, "Empty Stack"}
  """
  @spec pop(__MODULE__.t()) :: {:error, String.t()} | {:ok, any(), __MODULE__.t()}
  def pop(%__MODULE__{elements: []}), do: {:error, "Empty Stack"}
  def pop(%__MODULE__{elements: [top | rest]}), do: {:ok, top, %__MODULE__{elements: rest}}
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
  Check if stack is empty

  ## Examples
    iex> Stack.empty?(%Stack{elements: ["item one"]})
    false

    iex> Stack.empty?(%Stack{elements: []})
    true
  """
  def empty?(%__MODULE__{elements: [_head | _tail]}), do: false
  def empty?(%__MODULE__{elements: []}), do: true
```

## Conclusion
Stacks are a rudimentary data structure but they are used constantly in
programming.
They help us allocate memory, parse code, add ways to undo actions, and much
more.

Implementing a Stack in Elixir is really just sugar for built in Elixir lists
but was a nice way to get acquainted with pattern matching and structs.

[Click for the full
implementation](https://github.com/SamuelWillis/algorithms/blob/main/elixir/lib/data_structures/stack.ex).

[And here is my full set of Stack
notes](https://github.com/SamuelWillis/algorithms/tree/main/notes/data-structures/stack).
