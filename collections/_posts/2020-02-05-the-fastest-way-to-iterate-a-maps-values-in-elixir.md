---
title: "The fastest way to iterate a Map’s values in Elixir"
layout: post
authors: ['tyler']
categories: ["Programming", 'Elixir']
---

Say you have an [Elixir](https://elixir-lang.org) [`Map`](https://hexdocs.pm/elixir/Map.html). What’s the fastest way to iterate the values?

The candidates to consider are:

*   Use `Enum.map/2` and just pick out the values, ignoring the key
*   Use `Map.values/1` and pipe the resulting list into `Enum.map/2`

I put together the following [Benchfella](https://github.com/alco/benchfella) microbenchmark:

```elixir
defmodule BasicBench do
  use Benchfella
 
  @test_map Map.new(1..10_000, fn k -> {k, k * k} end)
 
  bench "Iterate with Enum.map (anonymous function)" do
    Enum.map(@test_map, fn {_k, v} -> :math.sqrt(v) end)
  end
 
  bench "Iterate with Enum.map (capture)" do
    Enum.map(@test_map, &:math.sqrt(elem(&1, 1)))
  end
 
  bench "Iterate Map.values (anonymous function)" do
    @test_map
    |> Map.values()
    |> Enum.map(fn v -> :math.sqrt(v) end)
  end
 
  bench "Iterate Map.values (capture)" do
    @test_map
    |> Map.values()
    |> Enum.map(&:math.sqrt/1)
  end
end
```

Here’s what I got running on my 2019 MacBook Pro:

    benchmark name                                iterations average time
    Iterate Map.values (anonymous function)             5000 385.89 µs/op
    Iterate Map.values (capture)                        5000 389.60 µs/op
    Iterate with Enum.map (capture)                     1000 1022.27 µs/op
    Iterate with Enum.map (anonymous function)          1000 1026.64 µs/op

This was surprising to me. Contrary to my intuitions about how `Map` would be implemented under the hood, grabbing the list of `Map.values` list was about 2.5× faster than using `Enum.map` on the key-value pairs directly.

Also surprising: there’s no difference between the anonymous function and the capture syntax. (I kind of suspected the capture syntax would get optimized differently, but successive runs of the test flip back and forth between the two variants winning.)

**TL;DR: Use `Map.values/1`.**





