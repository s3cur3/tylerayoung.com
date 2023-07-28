---
title: "TIL: Elixir's <code>match?/2</code> turns a match result into a predicate"
layout: post
authors: ['tyler']
categories: ["Programming", "Elixir", "Today I Learned (TIL)"]
excerpt_full: true
---

TIL about Elixir's [`Kernel.match?/2`](https://hexdocs.pm/elixir/Kernel.html#match?/2).

```elixir
iex> list = [a: 1, b: 2, a: 3]
iex> Enum.filter(list, &match?({:a, _}, &1))
[a: 1, a: 3]
```

So many times I've hacked around not knowing about this. 🤯

Where this gets really useful is in tests where you want to assert *part* of the shape of a map. For instance:

```elixir
assert Enum.count(
         results,
         &match?(%{id: ^my_id, inserted_at: ^time}, &1)
       ) == 1
```
