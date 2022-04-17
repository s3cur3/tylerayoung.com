---
title: Enum.slide/3 is coming in Elixir 1.13
categories: ['Programming', 'Functional Programming', 'Elixir']
authors: ['tyler']
layout: post
---

Earlier this week, José Valim merged my first PR to the Elixir standard library. (Woohoo!) I figured it was worth creating a blog post to explain what a “slide” is, and why it might be valuable.

The classic use case is this: Suppose you have a list of to-do items, which the user has ordered by priority:

1.  Apply to college
2.  Brush the dog
3.  Change the car’s oil
4.  Deliver flowers
5.  Exchange gifts

(Notice the first character of those items are in alphabetical order.)

A “slide” or “rotate” occurs when the user selects some number of elements and drags them to a new place in the list. Let’s say they selected items 3 & 4 from the preceding and dragged them above item 2. When they release the mouse, the new order should be:

1.  Apply to college
2.  Change the car’s oil
3.  Deliver flowers
4.  Brush the dog
5.  Exchange gifts

Without the named algorithm, the easiest way to do this is to make 3 calls to `Enum.slice/3` (one at the insertion point, one at the start of the selected range, and one at the end of the selected range), then rejoin your 4 chunks. It’s easy to get the index math wrong, and it’s even harder for readers of your code to grasp what’s going on… and it’s also substantially slower than [the implementation that’s in 1.13](https://hexdocs.pm/elixir/1.13.0-rc.0/Enum.html#slide/3).

[A number of other languages](https://twitter.com/code_report/status/1419900906062204939) have a rotate algorithm, though it’s still somewhat uncommon. I found [Dave Abrahams’ comments](https://forums.swift.org/t/proposal-implement-a-rotate-algorithm-equivalent-to-std-rotate-in-c/491/2) valuable when this was discussed for inclusion in Swift.

This is the kind of algorithm that, when I first heard about it, I didn’t have any immediate use cases, but merely by being aware of its existence, I started seeing over time.

Most recently, this came up when working with day-of-the-week + time of day time ranges—think Cron scheduling, like “Mondays from 4 to 5 pm and Thursdays from 12 to 3:30 am.” Given a sorted list of the `{start_day_time, end_day_time}` pairs, how do you get the _inverse_ time ranges—that is, how do you get to the list of all times not included in that schedule? It’s easy with a rotate:

```elixir
def invert(day_time_range_pairs) do
  day_time_range_pairs
  |> Enum.flat_map(&[&1, &2])
  # Move the first element to the tail
  |> Enum.rotate(0, -1)
  # Reconstruct [start, end] pairs
  |> Enum.chunk_every(2)
  |> Enum.map(fn [start, stop] -> {start, stop} end)
end
```

I’m hoping bringing this to the standard library will support more clear communication in the code when this sort of thing comes up.
