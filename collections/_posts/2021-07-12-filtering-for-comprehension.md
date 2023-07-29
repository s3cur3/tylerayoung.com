---
title: "TIL Elixir's <code>for</code> comprehension supports filtering"
layout: post
authors: ['tyler']
categories: ["Programming", "Elixir", "Today I Learned (TIL)"]
excerpt_full: true
---

TIL about the filtering support in Elixir's `for` comprehensions ("don't call it a loop"):

```elixir
# Create a record every 5 seconds for a day
for seconds <- 0..(60 * 60 * 24),
    rem(seconds, 5) == 0 do
  create_record_at_offset(seconds)
end
```

For more on the amazing superpowers of the `for` comprehension, see Mitch Hanberg's [Comprehensive Guide to Elixir's List Comprehension](https://www.mitchellhanberg.com/the-comprehensive-guide-to-elixirs-for-comprehension/).

