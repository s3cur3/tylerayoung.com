---
title: "TIL Elixir's || supports nil fallback"
layout: post
authors: ['tyler']
categories: ["Programming", "Elixir", "Today I Learned (TIL)"]
excerpt_full: true
---

TIL Elixir can do `nil || something` (evaluating to `something`) just like the Javascript trick:

```elixir
default_value = 123
my_value = my_map[:some_field] || default_value
```

If `:some_field` is present & non-`nil`, `my_value` will be assigned the value from the map; otherwise it will fall back to `default_value`.

(Note that this works with the `||` operator, but not `or`—if this is as surprising to you as it was to me, see [TIL about the differences between Elixir's boolean operators](/2022/04/12/boolean-operators/). 😅)

